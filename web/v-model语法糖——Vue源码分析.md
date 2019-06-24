v-model语法糖
---


首先第一点就是如何知道关于`v-modal`的指令的，这要从$mount阶段的AST树（建议重点掌握，这块是很多技能的核心）

如一下的代码片段

```html
<ul :class="bindCls" class="list" v-if="isShow">
    <li v-for="(item,index) in data" @click="clickItem(index)">{{item}}:{{index}}</li>
</ul>
```
解析成AST结构代码描述如下：

```javascript
ast = {
  'type': 1,
  'tag': 'ul',
  'attrsList': [],
  'attrsMap': {
    ':class': 'bindCls',
    'class': 'list',
    'v-if': 'isShow'
  },
  'if': 'isShow',
  'ifConditions': [{
    'exp': 'isShow',
    'block': // ul ast element
  }],
  'parent': undefined,
  'plain': false,
  'staticClass': 'list',
  'classBinding': 'bindCls',
  'children': [{
    'type': 1,
    'tag': 'li',
    'attrsList': [{
      'name': '@click',
      'value': 'clickItem(index)'
    }],
    'attrsMap': {
      '@click': 'clickItem(index)',
      'v-for': '(item,index) in data'
     },
    'parent': // ul ast element
    'plain': false,
    'events': {
      'click': {
        'value': 'clickItem(index)'
      }
    },
    'hasBindings': true,
    'for': 'data',
    'alias': 'item',
    'iterator1': 'index',
    'children': [
      'type': 2,
      'expression': '_s(item)+":"+_s(index)'
      'text': '{{item}}:{{index}}',
      'tokens': [
        {'@binding':'item'},
        ':',
        {'@binding':'index'}
      ]
    ]
  }]
}
```
以上就是AST树状结构，每一个节点都是一个 `ast element`,当然它还维护了父子关系，`parent`为父节点`children`为子节点

当然这只是示例，如果有`v-modal`还是会放置到`attrsMap`中。

好了，正文开始分析，大家记得Vue的$mount阶段最重要的环节之一：AST parse => optimize =>  generate

### 流程分析

看到了上面的代码输出就是在 AST parse阶段，其中在最后generate阶段有个解析指令的阶段

```javascript
function genDirectives (el: ASTElement, state: CodegenState): string | void {
  const dirs = el.directives
  if (!dirs) return
  let res = 'directives:['
  let hasRuntime = false
  let i, l, dir, needRuntime
  for (i = 0, l = dirs.length; i < l; i++) {
    dir = dirs[i]
    needRuntime = true
    const gen: DirectiveFunction = state.directives[dir.name]
    if (gen) {
      // compile-time directive that manipulates AST.
      // returns true if it also needs a runtime counterpart.
      needRuntime = !!gen(el, dir, state.warn)
    }
    if (needRuntime) {
      hasRuntime = true
      res += `{name:"${dir.name}",rawName:"${dir.rawName}"${
        dir.value ? `,value:(${dir.value}),expression:${JSON.stringify(dir.value)}` : ''
      }${
        dir.arg ? `,arg:"${dir.arg}"` : ''
      }${
        dir.modifiers ? `,modifiers:${JSON.stringify(dir.modifiers)}` : ''
      }},`
    }
  }
  if (hasRuntime) {
    return res.slice(0, -1) + ']'
  }
}

```

其中`const gen: DirectiveFunction = state.directives[dir.name]`根据不同平台获取不同的方法，那么对应的model方法

```javascript
function model (
  el: ASTElement,
  dir: ASTDirective,
  _warn: Function
): ?boolean {
  warn = _warn
  const value = dir.value
  const modifiers = dir.modifiers
  const tag = el.tag
  const type = el.attrsMap.type

  if (process.env.NODE_ENV !== 'production') {
    const dynamicType = el.attrsMap['v-bind:type'] || el.attrsMap[':type']
    if (tag === 'input' && dynamicType) {
      warn(
        `<input :type="${dynamicType}" v-model="${value}">:\n` +
        `v-model does not support dynamic input types. Use v-if branches instead.`
      )
    }
    // inputs with type="file" are read only and setting the input's
    // value will throw an error.
    if (tag === 'input' && type === 'file') {
      warn(
        `<${el.tag} v-model="${value}" type="file">:\n` +
        `File inputs are read only. Use a v-on:change listener instead.`
      )
    }
  }

  if (el.component) {
    genComponentModel(el, value, modifiers)
    // component v-model doesn't need extra runtime
    return false
  } else if (tag === 'select') {
    genSelect(el, value, modifiers)
  } else if (tag === 'input' && type === 'checkbox') {
    genCheckboxModel(el, value, modifiers)
  } else if (tag === 'input' && type === 'radio') {
    genRadioModel(el, value, modifiers)
  } else if (tag === 'input' || tag === 'textarea') {
    genDefaultModel(el, value, modifiers)
  } else if (!config.isReservedTag(tag)) {
    genComponentModel(el, value, modifiers)
    // component v-model doesn't need extra runtime
    return false
  } else if (process.env.NODE_ENV !== 'production') {
    warn(
      `<${el.tag} v-model="${value}">: ` +
      `v-model is not supported on this element type. ` +
      'If you are working with contenteditable, it\'s recommended to ' +
      'wrap a library dedicated for that purpose inside a custom component.'
    )
  }

  // ensure runtime directive metadata
  return true
}
```

在这里我们会分析`genDefaultModel`这个模块，当然关于组件的模块我们稍后分析。

### 表单元素的v-model

关于`genDefaultModel`模块，我们一起来看看如下代码片段

```javascript
function genDefaultModel (
  el: ASTElement,
  value: string,
  modifiers: ?ASTModifiers
): ?boolean {
  const type = el.attrsMap.type
  const { lazy, number, trim } = modifiers || {}
  const needCompositionGuard = !lazy && type !== 'range'
  const event = lazy
    ? 'change'
    : type === 'range'
      ? RANGE_TOKEN
      : 'input'

  let valueExpression = '$event.target.value'
  if (trim) {
    valueExpression = `$event.target.value.trim()`
  }
  if (number) {
    valueExpression = `_n(${valueExpression})`
  }

  let code = genAssignmentCode(value, valueExpression)
  if (needCompositionGuard) {
    code = `if($event.target.composing)return;${code}`
  }

  addProp(el, 'value', `(${value})`)
  addHandler(el, event, code, null, true)
  if (trim || number) {
    addHandler(el, 'blur', '$forceUpdate()')
  }
}

```

这里我们看到`genAssignment`的方法

```javascript
function genAssignmentCode (
  value: string,
  assignment: string
): string {
  const modelRs = parseModel(value)
  if (modelRs.idx === null) {
    return `${value}=${assignment}`
  } else {
    return `$set(${modelRs.exp}, ${modelRs.idx}, ${assignment})`
  }
}
```
到了这里，通过`${value}=${assignment}`最终会生成`message=$event.target.value`。然后执行后续的业务逻辑`needCompositionGuard`最终执行结果的代码为
`if($event.target.composing)return;message=$event.target.value`。

执行完毕后后续有添加了两个行为

```javascript
  addProp(el, 'value', `(${value})`)
  addHandler(el, event, code, null, true)
```

为何说`v-model`是语法糖，这里就是原因和精髓，它帮你实现了动态`表单元素`上绑定了`value`，同时添加了事件，用代码演示如下：

```javascript
    <input
      v-bind:value="message"
      v-on:input="message=$event.target.value">
```

###  组件的v-model

找了个Demo，比较具有说服力

```javascript
let Child = {
  template: '<div>'
  + '<input :value="value" @input="updateValue" placeholder="edit me">' +
  '</div>',
  props: ['value'],
  methods: {
    updateValue(e) {
      this.$emit('input', e.target.value)
    }
  }
}

let vm = new Vue({
  el: '#app',
  template: '<div>' +
  '<child v-model="message"></child>' +
  '<p>Message is: {{ message }}</p>' +
  '</div>',
  data() {
    return {
      message: ''
    }
  },
  components: {
    Child
  }
})
```
以上的语法大家不能再熟悉了，父组件引用了`child`并且使用了`v-model`关联了数据`message`,
再看子组件的内部定义了`props`和`methods`。这是必要条件哦

为什么会这么定义呢？我们一起来看看源码怎么解释的

```javascript
else if (!config.isReservedTag(tag)) {
    genComponentModel(el, value, modifiers)
    // component v-model doesn't need extra runtime
    return false
}
```
这段业务逻辑是描述如果是组件的情况下，如何去解析`v-model`

那么`genComponentModel`的代码如下

```javascript
function genComponentModel (
  el: ASTElement,
  value: string,
  modifiers: ?ASTModifiers
): ?boolean {
  const { number, trim } = modifiers || {}

  const baseValueExpression = '$$v'
  let valueExpression = baseValueExpression
  if (trim) {
    valueExpression =
      `(typeof ${baseValueExpression} === 'string'` +
        `? ${baseValueExpression}.trim()` +
        `: ${baseValueExpression})`
  }
  if (number) {
    valueExpression = `_n(${valueExpression})`
  }
  const assignment = genAssignmentCode(value, valueExpression)

  el.model = {
    value: `(${value})`,
    expression: `"${value}"`,
    callback: `function (${baseValueExpression}) {${assignment}}`
  }
}
```

其实这个就是生成一段代码`el.model`:

```javascript
el.model = {
  callback:'function ($$v) {message=$$v}',
  expression:'"message"',
  value:'(message)'
}
```

我们再往前回顾一下`genData`的方法

```javascript
genData (el: ASTElement, state: CodegenState): string {
  let data = '{'

  // directives first.
  // directives may mutate the el's other properties before they are generated.
  const dirs = genDirectives(el, state)
  if (dirs) data += dirs + ','

  // key
  if (el.key) {
    data += `key:${el.key},`
  }
  // ref
  if (el.ref) {
    data += `ref:${el.ref},`
  }
  if (el.refInFor) {
    data += `refInFor:true,`
  }
  // pre
  if (el.pre) {
    data += `pre:true,`
  }
  // record original tag name for components using "is" attribute
  if (el.component) {
    data += `tag:"${el.tag}",`
  }
  // module data generation functions
  for (let i = 0; i < state.dataGenFns.length; i++) {
    data += state.dataGenFns[i](el)
  }
  // attributes
  if (el.attrs) {
    data += `attrs:{${genProps(el.attrs)}},`
  }
  // DOM props
  if (el.props) {
    data += `domProps:{${genProps(el.props)}},`
  }
  // event handlers
  if (el.events) {
    data += `${genHandlers(el.events, false, state.warn)},`
  }
  if (el.nativeEvents) {
    data += `${genHandlers(el.nativeEvents, true, state.warn)},`
  }
  // slot target
  if (el.slotTarget) {
    data += `slot:${el.slotTarget},`
  }
  // scoped slots
  if (el.scopedSlots) {
    data += `${genScopedSlots(el.scopedSlots, state)},`
  }
  // component v-model
  if (el.model) {
    data += `model:{value:${
      el.model.value
    },callback:${
      el.model.callback
    },expression:${
      el.model.expression
    }},`
  }
  // inline-template
  if (el.inlineTemplate) {
    const inlineTemplate = genInlineTemplate(el, state)
    if (inlineTemplate) {
      data += `${inlineTemplate},`
    }
  }
  data = data.replace(/,$/, '') + '}'
  // v-bind data wrap
  if (el.wrapData) {
    data = el.wrapData(data)
  }
  // v-on data wrap
  if (el.wrapListeners) {
    data = el.wrapListeners(data)
  }
  return data
}
```

在以上`el.model`为true的时候触发，最终的`render`代码：

```javascript
with(this){
  return _c('div',[_c('child',{
    model:{
      value:(message),
      callback:function ($$v) {
        message=$$v
      },
      expression:"message"
    }
  }),
  _c('p',[_v("Message is: "+_s(message))])],1)
}
```

同时在创建组件`vnode`阶段，会执行`createComponent`函数。里面也会对`data.model`进行处理

```javascript
if (isDef(data.model)) {
   transformModel(Ctor.options, data)
 }
```

那么其中`transformModel`

```javascript
function transformModel (options, data: any) {
  const prop = (options.model && options.model.prop) || 'value'
  const event = (options.model && options.model.event) || 'input'
  ;(data.props || (data.props = {}))[prop] = data.model.value
  const on = data.on || (data.on = {})
  if (isDef(on[event])) {
    on[event] = [data.model.callback].concat(on[event])
  } else {
    on[event] = data.model.callback
  }
}
```

这里的业务逻辑就是给添加`value`和事件`on`，基本就是如下：

```javascript
data.props = {
  value: (message),
}
data.on = {
  input: function ($$v) {
    message=$$v
  }
}
```
转换成组件的形式如下：

```javascript
let vm = new Vue({
  el: '#app',
  template: '<div>' +
  '<child :value="message" @input="message=arguments[0]"></child>' +
  '<p>Message is: {{ message }}</p>' +
  '</div>',
  data() {
    return {
      message: ''
    }
  },
  components: {
    Child
  }
})
```

很多人一看，啊哦，就是普通的父子组件通讯吗，所以说组件的`v-model`也是语法糖了。

当然需要看到子组件和事件名称可以支持自定义

```javascript
  const prop = (options.model && options.model.prop) || 'value'
  const event = (options.model && options.model.event) || 'input'
```

来个变种的例子

```javascript
let Child = {
  template: '<div>'
  + '<input :value="msg" @input="updateValue" placeholder="edit me">' +
  '</div>',
  props: ['msg'],
  model: {
    prop: 'msg',
    event: 'change'
  },
  methods: {
    updateValue(e) {
      this.$emit('change', e.target.value)
    }
  }
}

let vm = new Vue({
  el: '#app',
  template: '<div>' +
  '<child v-model="message"></child>' +
  '<p>Message is: {{ message }}</p>' +
  '</div>',
  data() {
    return {
      message: ''
    }
  },
  components: {
    Child
  }
})
```

好了，到此处就分析完了。觉得不错的请点赞哦~~







Vue之静态节点标记知识点
---

### 原因

最近面试，聊了一个话题，关于静态节点的知识，从AST => optimize => code gen 的流程知识大概了解了一下，
涉及到这方面具体执行流程没有关心，所以当时比较尴尬，目前会看一下，然后分享给大家

### optimize

我们还记得模版 `template` 经过 `parse` 过程后，会输出生成AST书，之后就是优化阶段， `optimize`，优化的原因是
Vue是数据驱动，响应式的，但是我们模版中很多数据都是静态节点，非响应式的，也有数据是首次渲染后不会放生变了的，拿这些数据
是不是在 `patch` 过程中跳过对比节省时间了呢？答案当然是肯定的。那么就看看如何做的吧。

首先是找到 `src/compiler/optimizer.js` 文件。然后找到对应的代码片段

```javascript
/**
 * Goal of the optimizer: walk the generated template AST tree
 * and detect sub-trees that are purely static, i.e. parts of
 * the DOM that never needs to change.
 *
 * Once we detect these sub-trees, we can:
 *
 * 1. Hoist them into constants, so that we no longer need to
 *    create fresh nodes for them on each re-render;
 * 2. Completely skip them in the patching process.
 */
export function optimize (root: ?ASTElement, options: CompilerOptions) {
  if (!root) return
  isStaticKey = genStaticKeysCached(options.staticKeys || '')
  isPlatformReservedTag = options.isReservedTag || no
  // first pass: mark all non-static nodes.
  markStatic(root)
  // second pass: mark static roots.
  markStaticRoots(root, false)
}
```
 看到了上面的代码片段了吧，其实就是做 `markStatic` 标记静态节点和 `markStaticRoots` 标记静态根节点
 
 具体涉及到内部的如何标记的代码请自行到对应的Vue源码中查看，只分析如何标记的基本原理
 
 + 首先会执行 `isStatic` 方法，判断AST是否是静态的，包含表达式、`v-if`、`v-for`等其他指令都是非静态的。如果是普通元素、纯文本元素、`v-pre`、`v-once`等（自己意会去吧）都是静态节点。
 
 + 如果递归遍历父节点中所有子节点的 `markStatic` 静态属性的话发现全部子节点都是静态的，则在父节点标记 `node.staticRoot = true` 静态根节点
 
 我在网上找到了一段优化后的代码贴出来，就是通过optimize优化之后
 
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
  'static': false,
  'staticRoot': false,
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
    'static': false,
    'staticRoot': false,
    'children': [
      'type': 2,
      'expression': '_s(item)+":"+_s(index)'
      'text': '{{item}}:{{index}}',
      'tokens': [
        {'@binding':'item'},
        ':',
        {'@binding':'index'}
      ],
      'static': false
    ]
  }]
}
```

哈哈，到了最为关键的生成可执行的函数阶段速成 `code gen` 阶段，那么这块一起来看看吧。代码流程太多，只关心静态代码转换对比阶段如何处理的，其他的省略了...


首先是补习一下一些常用的简称

```javascript
vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false)

export function installRenderHelpers (target: any) {
  target._o = markOnce
  target._n = toNumber
  target._s = toString
  target._l = renderList
  target._t = renderSlot
  target._q = looseEqual
  target._i = looseIndexOf
  target._m = renderStatic
  target._f = resolveFilter
  target._k = checkKeyCodes
  target._b = bindObjectProps
  target._v = createTextVNode
  target._e = createEmptyVNode
  target._u = resolveScopedSlots
  target._g = bindObjectListeners
}
```

具体以下代码分析的时候可能会遇到，直接查阅即可

那就直接看这段

```javascript
const code = generate(ast, options)
```

没错这就是开始执行的代码了，我们根据代码` const code = ast ? genElement(ast, state) : '_c("div")'`跳进去找到 `genElement`
以下就是 `genElement` 代码片段

```javascript
export function genElement (el: ASTElement, state: CodegenState): string {
  if (el.staticRoot && !el.staticProcessed) {
    return genStatic(el, state)
  } else if (el.once && !el.onceProcessed) {
    return genOnce(el, state)
  } else if (el.for && !el.forProcessed) {
    return genFor(el, state)
  } else if (el.if && !el.ifProcessed) {
    return genIf(el, state)
  } else if (el.tag === 'template' && !el.slotTarget) {
    return genChildren(el, state) || 'void 0'
  } else if (el.tag === 'slot') {
    return genSlot(el, state)
  } else {
    // component or element
    let code
    if (el.component) {
      code = genComponent(el.component, el, state)
    } else {
      const data = el.plain ? undefined : genData(el, state)

      const children = el.inlineTemplate ? null : genChildren(el, state, true)
      code = `_c('${el.tag}'${
        data ? `,${data}` : '' // data
      }${
        children ? `,${children}` : '' // children
      })`
    }
    // module transforms
    for (let i = 0; i < state.transforms.length; i++) {
      code = state.transforms[i](el, code)
    }
    return code
  }
}
```
其他的条件判断可以自行根据断点查看，一起直接进入本章的静态属性分析环节`genStatic`的方法，如果是全
静态的根节点

```javascript
// hoist static sub-trees out
function genStatic (el: ASTElement, state: CodegenState): string {
  el.staticProcessed = true
  state.staticRenderFns.push(`with(this){return ${genElement(el, state)}}`)
  return `_m(${state.staticRenderFns.length - 1}${el.staticInFor ? ',true' : ''})`
}
```
呃，直接返回，不用做那么多的优化指令等条件分析，是不是很快。那么有的同学会问针对于`static`节点没有执行流程吗？其实只要有子节点`children`
和子节点都是`static`状态下才会标记`rootStatic 为 true` 所以`static`只是辅助标记的对象，真正做判断的就是每一个有子节点（就一个文本除外）都是静态的，那么
才会有当前节点为 `rootStatic = true` 并且codeGen阶段很快很顺利的生成render函数。好了，关于静态节点这些知识点分析到这里。
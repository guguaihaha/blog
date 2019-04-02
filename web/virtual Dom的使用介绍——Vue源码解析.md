virtual Dom的实现方式
---

什么是虚拟Dom结构，那先说说什么是真实Dom结构。

现实中`body`,`div`,`p`等标签都是真实的Dom结构的节点，这些结构的层级和组织成了真实的Dom树，当然这个结构不仅仅有这些标签组成
还有对应的属性和方法。

我们一起看看div的属性有多少

```text
align title lang translate dir dataset hidden tabIndex accessKey draggable spellcheck autocapitalize contentEditable isContentEditable inputMode offsetParent offsetTop offsetLeft offsetWidth offsetHeight style innerText outerText oncopy oncut onpaste onabort onblur oncancel oncanplay oncanplaythrough onchange onclick onclose oncontextmenu oncuechange ondblclick ondrag ondragend ondragenter ondragleave ondragover ondragstart ondrop ondurationchange onemptied onended onerror onfocus oninput oninvalid onkeydown onkeypress onkeyup onload onloadeddata onloadedmetadata onloadstart onmousedown onmouseenter onmouseleave onmousemove onmouseout onmouseover onmouseup onmousewheel onpause onplay onplaying onprogress onratechange onreset onresize onscroll onseeked onseeking onselect onstalled onsubmit onsuspend ontimeupdate ontoggle onvolumechange onwaiting onwheel onauxclick ongotpointercapture onlostpointercapture onpointerdown onpointermove onpointerup onpointercancel onpointerover onpointerout onpointerenter onpointerleave onselectstart onselectionchange nonce click focus blur namespaceURI prefix localName tagName id className classList slot attributes shadowRoot assignedSlot innerHTML outerHTML scrollTop scrollLeft scrollWidth scrollHeight clientTop clientLeft clientWidth clientHeight attributeStyleMap onbeforecopy onbeforecut onbeforepaste onsearch previousElementSibling nextElementSibling children firstElementChild lastElementChild childElementCount onwebkitfullscreenchange onwebkitfullscreenerror setPointerCapture releasePointerCapture hasPointerCapture hasAttributes getAttributeNames getAttribute getAttributeNS setAttribute setAttributeNS removeAttribute removeAttributeNS hasAttribute hasAttributeNS toggleAttribute getAttributeNode getAttributeNodeNS setAttributeNode setAttributeNodeNS removeAttributeNode closest matches webkitMatchesSelector attachShadow getElementsByTagName getElementsByTagNameNS getElementsByClassName insertAdjacentElement insertAdjacentText insertAdjacentHTML requestPointerLock getClientRects getBoundingClientRect scrollIntoView scrollIntoViewIfNeeded animate computedStyleMap before after replaceWith remove prepend append querySelector querySelectorAll webkitRequestFullScreen webkitRequestFullscreen onfullscreenchange onfullscreenerror scroll scrollTo scrollBy createShadowRoot getDestinationInsertionPoints requestFullscreen ELEMENT_NODE ATTRIBUTE_NODE TEXT_NODE CDATA_SECTION_NODE ENTITY_REFERENCE_NODE ENTITY_NODE PROCESSING_INSTRUCTION_NODE COMMENT_NODE DOCUMENT_NODE DOCUMENT_TYPE_NODE DOCUMENT_FRAGMENT_NODE NOTATION_NODE DOCUMENT_POSITION_DISCONNECTED DOCUMENT_POSITION_PRECEDING DOCUMENT_POSITION_FOLLOWING DOCUMENT_POSITION_CONTAINS DOCUMENT_POSITION_CONTAINED_BY DOCUMENT_POSITION_IMPLEMENTATION_SPECIFIC nodeType nodeName baseURI isConnected ownerDocument parentNode parentElement childNodes firstChild lastChild previousSibling nextSibling nodeValue textContent hasChildNodes getRootNode normalize cloneNode isEqualNode isSameNode compareDocumentPosition contains lookupPrefix lookupNamespaceURI isDefaultNamespace insertBefore appendChild replaceChild removeChild addEventListener removeEventListener dispatchEvent 
```

是不是很多，真实的Dom元素属性很是庞大，但是实际去操作这些都会触发浏览器更新Dom，会产生性能问题。

而 Virtual DOM 就是用一个原生的 JS 对象去描述一个 DOM 节点，所以它比创建一个 DOM 的代价要小很多。在 Vue.js 中，Virtual DOM 是用 VNode 这么一个 Class 去描述，它是定义在 `src/core/vdom/vnode.js` 中的。

```javascript
export default class VNode {
  tag: string | void;
  data: VNodeData | void;
  children: ?Array<VNode>;
  text: string | void;
  elm: Node | void;
  ns: string | void;
  context: Component | void; // rendered in this component's scope
  functionalContext: Component | void; // only for functional component root nodes
  key: string | number | void;
  componentOptions: VNodeComponentOptions | void;
  componentInstance: Component | void; // component instance
  parent: VNode | void; // component placeholder node
  raw: boolean; // contains raw HTML? (server only)
  isStatic: boolean; // hoisted static node
  isRootInsert: boolean; // necessary for enter transition check
  isComment: boolean; // empty comment placeholder?
  isCloned: boolean; // is a cloned node?
  isOnce: boolean; // is a v-once node?
  asyncFactory: Function | void; // async component factory function
  asyncMeta: Object | void;
  isAsyncPlaceholder: boolean;
  ssrContext: Object | void;

  constructor (
    tag?: string,
    data?: VNodeData,
    children?: ?Array<VNode>,
    text?: string,
    elm?: Node,
    context?: Component,
    componentOptions?: VNodeComponentOptions,
    asyncFactory?: Function
  ) {
    this.tag = tag
    this.data = data
    this.children = children
    this.text = text
    this.elm = elm
    this.ns = undefined
    this.context = context
    this.functionalContext = undefined
    this.key = data && data.key
    this.componentOptions = componentOptions
    this.componentInstance = undefined
    this.parent = undefined
    this.raw = false
    this.isStatic = false
    this.isRootInsert = true
    this.isComment = false
    this.isCloned = false
    this.isOnce = false
    this.asyncFactory = asyncFactory
    this.asyncMeta = undefined
    this.isAsyncPlaceholder = false
  }

  // DEPRECATED: alias for componentInstance for backwards compat.
  /* istanbul ignore next */
  get child (): Component | void {
    return this.componentInstance
  }
}
```

***代码 1.1***

呃，看似代码量很多，但是内容和实际使用与真实DOM还是少很多的。因为常用的就那么几个，你全定义了也没啥作用啊。不过Vue的Virtual Dom是借鉴了  [snabbdom](https://github.com/snabbdom/snabbdom)。这个大家可以简单看一下，在这基础上
Vue还做了很多改进和适配。

对了，这就是virtual Dom的核心模块Vnode的代码片段，当然这还需要有一些`creat`、`update`、`patch`等方法。后续文章会有介绍。
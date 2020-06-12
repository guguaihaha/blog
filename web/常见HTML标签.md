常见HTML标签
---

今天分享一个常见的常量识别的方法，只是记录一下，方便后续使用

## 首先关于页面头位置的声明

```javascript
export const namespaceMap = {
  svg: 'http://www.w3.org/2000/svg',
  math: 'http://www.w3.org/1998/Math/MathML'
}
```

## 关于HTML元素的标签有哪些呢？

```javascript
export const isHTMLTag =
  'html,body,base,head,link,meta,style,title,' +
  'address,article,aside,footer,header,h1,h2,h3,h4,h5,h6,hgroup,nav,section,' +
  'div,dd,dl,dt,figcaption,figure,picture,hr,img,li,main,ol,p,pre,ul,' +
  'a,b,abbr,bdi,bdo,br,cite,code,data,dfn,em,i,kbd,mark,q,rp,rt,rtc,ruby,' +
  's,samp,small,span,strong,sub,sup,time,u,var,wbr,area,audio,map,track,video,' +
  'embed,object,param,source,canvas,script,noscript,del,ins,' +
  'caption,col,colgroup,table,thead,tbody,td,th,tr,' +
  'button,datalist,fieldset,form,input,label,legend,meter,optgroup,option,' +
  'output,progress,select,textarea,' +
  'details,dialog,menu,menuitem,summary,' +
  'content,element,shadow,template,blockquote,iframe,tfoot'
```

个人觉得很全了，可以直接拿来使用了，当然这些还是精挑细选的，具体方法``

## 关于SVG的节点有哪些

```javascript
export const isSVG = 
  'svg,animate,circle,clippath,cursor,defs,desc,ellipse,filter,font-face,' +
  'foreignObject,g,glyph,image,line,marker,mask,missing-glyph,path,pattern,' +
  'polygon,polyline,rect,switch,symbol,text,textpath,tspan,use,view'
```

## 关于输入框input的类型

```javascript
var inputType = 'text,number,password,search,email,tel,url'
```

## 键盘按键参考表

```text

字母和数字键的键码值(keyCode)
按键	键码	按键	键码	按键	键码	按键	键码
A	65	J	74	S	83	1	49
B	66	K	75	T	84	2	50
C	67	L	76	U	85	3	51
D	68	M	77	V	86	4	52
E	69	N	78	W	87	5	53
F	70	O	79	X	88	6	54
G	71	P	80	Y	89	7	55
H	72	Q	81	Z	90	8	56
I	73	R	82	0	48	9	57
数字键盘上的键的键码值(keyCode)	功能键键码值(keyCode)
按键	键码	按键	键码	按键	键码	按键	键码
0	96	8	104	F1	112	F7	118
1	97	9	105	F2	113	F8	119
2	98	*	106	F3	114	F9	120
3	99	+	107	F4	115	F10	121
4	100	Enter	108	F5	116	F11	122
5	101	-	109	F6	117	F12	123
6	102	.	110	 	 	 	 
7	103	/	111	 	 	 	 
控制键键码值(keyCode)
按键	键码	按键	键码	按键	键码	按键	键码
BackSpace	8	Esc	27	Right Arrow	39	-_	189
Tab	9	Spacebar	32	Dw Arrow	40	.>	190
Clear	12	Page Up	33	Insert	45	/?	191
Enter	13	Page Down	34	Delete	46	`~	192
Shift	16	End	35	Num Lock	144	[{	219
Control	17	Home	36	;:	186	|	220
Alt	18	Left Arrow	37	=+	187	]}	221
Cape Lock	20	Up Arrow	38	,<	188	'"	222
多媒体键码值(keyCode)
按键	键码	按键	键码	按键	键码	按键	键码
音量加	175	 	 	 	 	 	 
音量减	174	 	 	 	 	 	 
停止	179	 	 	 	 	 	 
静音	173	 	 	 	 	 	 
浏览器	172	 	 	 	 	 	 
邮件	180	 	 	 	 	 	 
搜索	170	 	 	 	 	 	 
收藏	171	 	 	 	 	 	 
```


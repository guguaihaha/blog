Javascript的常见算法
---

以下代码都是实测


### 首先是基本算法

最近经常有人问我一些基本算法，目前输出一些基于js的基本常见算法。经过测试，直接上代码：

```javascript
 var _ARY = [90,827,88,7,3,97,736,776,96,1,4,9,80,34,56,999,777,444,333,222,111,768,754]
    //默认算法
    _ARY.sort(function (a,b) {
        return a - b
    })

    //快速排序
    function quickSort(ret){
        if(ret.length <= 1){
            return ret
        }
        var left = [],
            right = [],
            mid = Math.floor((ret.length - 1)/2),
            _temp = ret.splice(mid,1)[0]
        for(var i = 0, l = ret.length; i < l; i++){
            if(ret[i] < _temp){
                left.push(ret[i])
            }else{
                right.push(ret[i])
            }
        }
        return quickSort(left).concat([_temp],quickSort(right))

    }

    //冒泡排序
    function popSort(ret){
        for(var i = 0, l = ret.length; i < l; i++){
            for(var j = 0; j < l; j++){
                if(ret[j] > ret[j + 1]){
                    var _temp = ret[j]
                    ret[j] = ret[j + 1]
                    ret[j + 1] = _temp
                }
            }
        }
    }
    //选择排序
    function choiceSort(ret){
        for(var i = 0, l = ret.length; i < l - 1; i++){
            var min = i
            for(var j = i + 1; j < l; j++){
                if(ret[min] > ret[j]){
                    min = j
                }
                if(i != min){
                    var _temp = ret[i]
                    ret[i] = ret[min]
                    ret[min] = _temp
                }
            }
        }
    }
    //插入排序
    function insertSort(ret){
        for(var i = 1, l = ret.length; i < l; i++){
            var  j = i - 1,
                 _temp = ret[i]
            while(j >= 0 && _temp < ret[j]){
                ret [j + 1] = ret [j]
                j--
            }
            ret[j + 1] = _temp
        }
    }

    //归并排序,利用分而治之
    function _merge(left,right){
        //治
        var result = []
            while(left.length > 0 && right.length > 0){
               if(left[0] < right[0]){
                   result.push(left.shift())
               }else{
                   result.push(right.shift())
               }
            }
            result = result.concat(left,right)
            return result
    }
    function mergeSort(ret){
        //分
        if(ret.length <=1){
           return ret
        }else{
            var mid = Math.floor((ret.length)/2),
                left = ret.slice(0,mid),
                right = ret.slice(mid)
            return _merge(mergeSort(left),mergeSort(right))
        }

    }
 
```

### 二分查找

```javascript
 //二分查找
    var binarySearch = function(ret, dest, start, end){
        var _s = start || 0,
            _e = end || ret.length - 1,
            _m = Math.floor((_s + _e)/2),
            _mn = ret[_m]
        if(_mn == dest){
            return `当前位置是数组下角标 ${_m}, 查找到的数值是 ${_mn}
数组是 [${ret}]`
        }
        if(dest < _mn){
            return binarySearch(ret, dest, 0, _m - 1)
        }else{
            return binarySearch(ret, dest, _m + 1, _e)
        }
        return false
    }
```

### 简单二叉树构建

```javascript
var _ARY = [90,827,88,7,3,97,736,776,96,1,4,9,80,34,56,999,777,444,333,222,111,768]
    var _length = _ARY.length;
    var _R = Math.ceil(Math.random() * (_length + 1))
    var _temp = _ARY.splice(_R,1)
    class Node {
        constructor(opt){
            this.data = opt.data
            this.left = opt.left || ''
            this.right = opt.right || ''
            this.parent = opt.parent || ''
            this.level = opt.level || 1
        }
        set(opt){
             let _this = this
             Object.keys(opt).forEach(function (item,index) {
                 _this[item] = opt[item]
             })
        }
    }


    class tree{
        constructor(){
            this.root = ''
            this.findFlag = ''
        }

        insertBlock(ENode,_root){
            let _this = this
            let ENroot = _root || this.root
            if(ENroot){
                ENode.set({parent:ENroot,level:ENroot.level + 1})
                if(ENroot.data < ENode.data){
                    //右侧树
                    if(ENroot.right){
                        _this.insertBlock(ENode,ENroot.right)
                    }else{
                        ENroot.right = ENode
                    }

                }else{
                    //左侧树
                    if(ENroot.left){
                        _this.insertBlock(ENode,ENroot.left)
                    }else{
                        ENroot.left = ENode
                    }
                }
                
            }else{
                this.root = ENode
            }
        }

        insert(data){
            let _tar = new Node({data})
            this.insertBlock(_tar)
        }

        _find(data,ENroot){
            let _this = this
            if(!ENroot || _this.findFlag){
                return
            }
            if(ENroot.data == data){
                _this.findFlag = ENroot
            }else{
                _this._find(data,ENroot.left)
                _this._find(data,ENroot.right)
            }
        }
        find(data){
            this.findFlag = ''
            this._find(data,this.root)
            return this.findFlag
        }
    }

    var _TREE = new tree()
    _TREE.insert(_temp)
    _ARY.forEach(function (item) {
        _TREE.insert(item)
    })
```

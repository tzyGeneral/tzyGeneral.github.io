# 基于webview的选择滑动控件



展示地址：http://zhaoziang.com/uiweb/selcontrol/list.htm

一、功能点：

1、点击控件，让小圆点跳到正确位置

2、拖动小圆点，让其跟随，并停在正确的位置

 

二、设计思路：

设计三个函数，分别是：

1、获取dom元素的当前位置。getPosition(_dom)

2、根据当前位置x，判断小圆点悬停位置的一个附近列表，该控件有6个可以悬停的地方。nearList(x)

3、让小圆点跳到正确位置的动画效果。moveTo(start,end)

 

三、代码实现：

1、获取dom元素的当前位置。getPosition(_dom)

输入：传入参数，dom元素。先获取到dom元素。

```javascript
   var _btn = document.getElementById("dragbtn");
    var _bar = document.getElementById("list_sel");
```

输出：该dom元素的位置。

tips:

1) 注意需要使用parseInt，将获取到的offsetLeft转换为数字。而在使用数字的时候，记得要加上+"px“，这样left属性才能正确识别哦。

2) 这里取元素的left值，使用dom.style.left，是取不到值的，应该使用offsetLeft。使用dom.style.left的场景，应该是left属性写在html里的，像这样<div style="left:10px"></div>

```js
//1、getPosition(_dom)
    function getPosition(dom){
        _dom = dom;
        var _x = parseInt(_dom.offsetLeft);
        return _x + 18;
    }
```

2、根据当前位置x，判断小圆点悬停位置的一个附近列表，该控件有6个可以悬停的地方。nearList(x)

这里是，只需要修改起点_start ，和间隔_space。就能获得整个附近列表的设计。这样子不用每次都去修改N个地方的参数。

tips:

大于>，小于<，等于=，大于等于>=，小于等于<=都是二元操作符。所以在if判断的时候，如果有两个以上的判断时，使用与&&符号来连接。就像：1<x && x<9。而不是写成1<x<9，这样是无法识别的。

```js
//2、nearList
    function nearList(x){
        var _movetox = 0;
        var _start = 78; //起点
        var _space = 56; //两点之间的间隔px        
        var _nearlist = [_start,
                        _start+_space,
                        _start+2*_space,
                        _start+3*_space,
                        _start+4*_space,
                        _start+5*_space];
        var _x = x;                    
        if(_x<=_nearlist[0]){
            _movetox = _nearlist[0];
        }
        else if(_nearlist[0]<_x && _x<=_nearlist[0]+_space/2){
            _movetox = _nearlist[0];
        }
        else if(_nearlist[1]-_space/2<_x && _x<=_nearlist[1]+_space/2){
            _movetox = _nearlist[1];
        }
        else if(_nearlist[2]-_space/2<_x && _x<=_nearlist[2]+_space/2){
            _movetox = _nearlist[2];
        }
        else if(_nearlist[3]-_space/2<_x && _x<=_nearlist[3]+_space/2){
            _movetox = _nearlist[3];
        }
        else if(_nearlist[4]-_space/2<_x && _x<=_nearlist[4]+_space/2){
            _movetox = _nearlist[4];
        }
        else if(_nearlist[5]-_space/2<_x && _x<=_nearlist[5]+_space/2){
            _movetox = _nearlist[5];
        }
        else if(_x>_nearlist[5]){
            _movetox = _nearlist[5];
        }
        return _movetox;
    }
```

3、让小圆点跳到正确位置的动画效果。moveTo(start,end)

起点_startX是dom元素的位置，终点_endX是根据附近列表选择的。动画效果，使用延时来修改left的值。

tips:

这里获得的位置，都是数字。所以需要加上+"px"。

```js
//3、moveTo
    function moveTo(start,end){
        var _startX = getPosition(_btn);
        var _endX = nearList(end);
        _btn.style.left = _endX - 76 + "px";
    }
```

 

四、效果事件：

1、给控件bar添加点击事件，让小圆点跳到正确的地方：

```
//点击bar的事件
    _bar.onclick = function(e){
        moveTo(0,e.clientX);
    }
```

2、给小圆点添加拖拽事件。

drag事件，是用onmousedown，onmouseup，onmousemove三个事件，加上一个isdrag开关来实现的。

具体实现方式是：

开关先关掉isdrag=false；

当鼠标按下onmousedown，触发开关isdrag=true；

然后开始拖动onmousemove；

当鼠标松开onmouseup时，关掉开关isdrag=false。

当鼠标松开onmouseup时，关掉开关isdrag=false。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//拖动btn的事件
    var _isdrag=false;
    var myX;
    var dobj;
    function movemouse(e){
        if (_isdrag)
        {
            dobj.style.left = tx + e.clientX - myX + "px";
            return false;
        }
    }
    function moving(e){
        var fobj = event.srcElement;
        if (fobj.id=="dragbtn"){
            _isdrag = true;
            dobj = fobj;
            tx = parseInt(dobj.style.left+0);
            myX = e.clientX;
            document.onmousemove = movemouse;
            return false;
        }
    }
    document.onmousedown = moving;
    document.onmouseup = function(e){
        _isdrag = false;
        moveTo(0,e.clientX);
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

\--------------------------------------------------------

PC版：

DOM结构设计，CSS样式，以及全部源代码，请到展示地址查看：

http://zhaoziang.com/uiweb/selcontrol/list.htm

 \--------------------------------------------------------

wap版：

 

手机浏览器上（指ios和android机器）的触屏事件，与PC上的鼠标事件，有所对应，分别为：

ontouchstart == onmousedown

ontouchend == onmouseup

ontouchmove == onmousemove

 

获取元素位置的方法也有所不同：

e.touches[0].pageX == e.clientX

 

针对上述两个不同，对于wap端，做了改进。

详情请用手机浏览器访问：http://zhaoziang.com/uiweb/selcontrol/index.htm
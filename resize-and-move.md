# 【Cocos Creator】 移动端中简单的移动和缩放（无边界）- 移动部分
> 只要是个地图，基本的移动和缩放几乎是必须的，甚至需要判断边界。这里先介绍基本的移动和缩放

## 1. 移动
> - 我要求手划过去，就能将地图移动一下。这样就得在需要移动的节点上去触发手指滑动事件。
> - 这些节点可以是本身节点（注意把需要在范围内），也可以是子节点（可以遍历子节点来绑定事件）。
> - 所有的移动也就将 原向量+移动向量（每次都很小） 赋给要移动的节点

```
//本身节点移动
@ccclass
export default class Move extends cc.Component{
    start(){
        this.node.on(cc.Node.EventType.TOUCH_MOVE, (e: cc.Event.EventTouch) => {
            this.node.position = this.node.position.add(e.getDelta());
        });
    }
}
//子节点移动
@ccclass
export default class ChildMove extends cc.Component{
    start(){
        this.node.children.map(c => {
            c.on(cc.Node.EventType.TOUCH_MOVE, (e: cc.Event.EventTouch) => {
                c.position = c.position.add(e.getDelta());
            });
        })
    }
}
//从编辑器传节点移动
@ccclass
export default class NodeMove extends cc.Component{
    @property(cc.Node)
    moveNode: cc.Node = null;   //这里不使用node, 以免无法获取绑定当前脚本的node

    start(){
        this.moveNode.on(cc.Node.EventType.TOUCH_MOVE, (e: cc.Event.EventTouch) => {
            this.moveNode.position = this.moveNode.position.add(e.getDelta());
        });
    }
}

```

## 2. 缩放
> - 我要求两只手同时按住，距离缩小，就进行缩放。同样需要使用TOUCH_MOVE事件，不过需要取两只手的id
> - 在事件中需要判断两手距离是否更近或更远

```
@ccclass
export default class Resize extends cc.Component{
    private _scale = 1;
    MIN_SCALE = 0.4;
    MAX_SCALE = 1;
    ONCE_SCALE = 0.05;//一次操作缩放倍数

    start(){
        let touchInfo: {tid: number, location: cc.Vec2}[] = [];

        //该事件用于对二点触控进行取样
        this.node.on(cc.Node.Event.TOUCH_START, (e: cc.Event.EventTouch) => {
            if(touchInfo.length >= 2){
                touchInfo.shift();
            }else{
                //如果有同样触控id，去除该触控id以便重新加入
                touchInfo = touchInfo.filter(t => t.tid !== e.getID());
            }
            touchInfo.push({tid: e.getID(), location: e.getLocation()})
        });

        //该事件用于二点触控距离判定
        this.node.on(cc.Node.Event.TOUCH_MOVE, (e: cc.Event.EventTouch) => {
            if(touchInfo.length >= 2){
                let nt = touchInfo.find(t => t.tid === e.getID());  //属于触控点的
                let pt = touchInfo.find(t => t.tid !== e.getID());  //不属于触控点的
                if(pt.tid !== nt.tid){//两个点id不一样才能因判断距离而进行缩放
                    if(getSquareDistance(pt.location, nt.location) 
                        > getSquareDistance(pt.location, e.getLocation())){//现距离小于原距离
                        this._scale -= Resize.ONCE_SCALE;
                    }else if(getSquareDistance(pt.location, nt.location) 
                        < getSquareDistance(pt.location, e.getLocation())){
                        this._scale += Resize.ONCE_SCALE;
                    }
                    doResize();//修改了scale大小记得resize处理
                }
            }
        });

        //如果手松开了，对应id的触控信息也要去掉
        let onTouchEnd = (e: cc.Event.EventTouch) => {
            touchInfo = touchInfo.filter(t => t.tid !== e.getID());
        }

        this.node.on(cc.Node.Event.TOUCH_END, onTouchEnd);

        this.node.on(cc.Node.Event.TOUCH_CANCEL, onTouchEnd);
    } 

    doResize(){
        this.node.scale = this._scale;
    }
}

function getSquareDistance(a: cc.Vec2, b: cc.Vec2){
    return (a.x - b.x) * (a.x - b.x) + (a.y - b.y) * (a.y - b.y);
}
```

    这里的缩放需要注意多点触控的操作判定，和两点距离的变化。

    实际代码比这个复杂点，还有其他的边界判断，但基本面就是这些。

[返回主页](./readme.md)
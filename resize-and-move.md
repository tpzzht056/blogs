# 【Cocos Creator】 移动端中简单的移动和缩放（无边界）- 移动部分
> 只要是个地图，基本的移动和缩放几乎是必须的，甚至需要判断边界。这里先介绍基本的移动和缩放

## 移动
> 我要求手划过去，就能将地图移动一下。这样就得在需要移动的节点上去触发手指滑动事件。
> 这些节点可以是本身节点（注意把需要在范围内），也可以是子节点（可以遍历子节点来绑定事件）。
> 所有的移动也就将 原向量+移动向量（每次都很小） 赋给要移动的节点

```
//本身节点移动
@ccclass
export default class Move{
    start(){
        this.node.on(cc.Node.EventType.TOUCH_MOVE, (e: cc.Event.EventTouch) => {
            this.node.position = this.node.position.add(e.getDelta());
        });
    }
}
//子节点移动
@ccclass
export default class ChildMove{
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
export default class NodeMove{
    @property(cc.Node)
    moveNode: cc.Node = null;   //这里不使用node, 以免无法获取绑定当前脚本的node

    start(){
        this.moveNode.on(cc.Node.EventType.TOUCH_MOVE, (e: cc.Event.EventTouch) => {
            this.moveNode.position = this.moveNode.position.add(e.getDelta());
        });
    }
}

```
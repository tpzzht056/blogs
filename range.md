# 【Cocos Creator】 边界的确定

- pochiko 2019-05-14

> 上期在 [移动和缩放](./resize-and-remove.md) 说到的场景都是简单的移动与缩放，连边界都不考虑。但是实际开发过程中，肯定都需要考虑边界的，要不然移动和缩放都能出黑边总是觉得有些糟糕。
> 这里简要说明实际边界的确定。

一、因素：
   1. 背景、布置的地面和建筑可以确定边界大概多大
   2. 地图的整体形状：比如棱形可以确定的边界只能是相应边中点连起来的长方形这种的；又比如长方形的边界就直接整体看待；又比如类长方形的拼图边界大概就是全拼图上下左右边界都适当减少若干px来确定。
   
二、实现
    本实现仅考虑长方形/长方形拼图边界的实现。基本实现就是遍历需要确定边界的建筑/背景，让边界逐步扩大的方式。
    
```typescript
@ccclass
export default class MapRange extends cc.Component{
    @property([cc.Node])
    itemNodes: cc.Node[] = [];  //用于确定边界的节点

    /** 根据可确定边界的节点来调整边界 */
    updateRange(){
        let [up, down, left, right] = [0, 0, 0, 0];
        ([up, down, left, right] = this.itemNodes.reduce((all, n) => {
            if(!all.length){
                return this.getRangeByNode(n);
            }
            return [
                up > all[0]? up: all[0],
                down < all[1]? down: all[1],
                left < all[2]? left: all[2],
                right > all[3]? right: all[3]
            ]
        }, []));
        //确定好边界后需要将边界转换为该node中的宽、高和对应anchor的范围来存储
        this.node.width = (right - left) / this.node.scaleX;
        this.node.height = (up - down) / this.node.scaleY;
        this.node.anchorX = (this.node.x - left) / (right - left);
        this.node.anchorY = (this.node.y - down) / (up - down);
    }

    /** 返回n对应该脚本绑定节点的范围[up, down, left, right] */
    getRangeByNode(n: cc.Node){
        //左上角的点
        let posLT = getPosBy(n, cc.v2(- n.width * n.anchorX, n.height * (1 - n.anchorY)), this.node);
        //右下角的点
        let porRD = getPosBy(n, cc.v2(n.width * (1 - n.anchorX), - n.height * n.anchorY), this.node);
        return [
            posLT.y,
            posRD.y,
            posLT.x,
            posRD.x
        ]
    }
}

/** 获取cur节点相对于by节点位置，未考虑by的父节点scale的情况 */
function getPosBy(cur: cc.Node, curpos: cc.Vec2, by: cc.Node){
    let curWorldPos = cur.convertToWorldSpaceAR(curpos);
    let byPos = by.convertToWorldSpaceAR(cc.v2(0, 0));
    return curWorldPos.add(byPos.neg()).mul(1 / by.scale);  //通过世界位置相减后，注意相对于by的缩放
}
```

    代码中，updateRange用于处理边界，这样，脚点绑定的边界均可根据需要确定边界的节点诸如背景、建筑等实现。本文中的边界确定也就简单写写，实际上比这个复杂（光输入总不可能直接用这种输入，更可能结合多个需要界定边界的父节点遍历子节点来制作），且不一定比这个实现更好。

    getRangeByNode的边界实现上，另一种方式就是直接拿n节点的宽/高和对应的缩放来确定n的边界，也是行得通的，不过没有本文实现更为准确，不过反正需要确定边界的节点都在脚本绑定节点子节点中，这点区别并不影响边界确定。
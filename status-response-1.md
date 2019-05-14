# 【Cocos Creator】切换状态立即响应组件触发 - 1
- pochiko 2019-05-13

比如，简易的狗动画根据状态来切换
```typescript
@ccclass
export default class DogAni extends cc.Component{
    @property(cc.Animation)
    dogAni: cc.Animation = null;

    private _status = 0;

    public get status(){
        return this._status;
    }

    public set status(s: number){
        this._status = s;
        updateStatus();
    }

    updateStatus(){
        dogAni.play(`dog_${this._status}`);
    }
}
```
将原来的状态字段加上getter/setter，然后在setter中调用状态更新函数，去切换动画。

但是，有时候，我并不希望在设置相同状态时立即重新播放动画，那就这样改
```typescript
    ... //代码的前面部分
    public set status(s: number){
        if(this._status !== s){
            this._status = s;
            updateStatus();
        }
    }
    ... //代码的后面部分
```
将要改动的部分加上前置条件

[返回主页](./readme.md)
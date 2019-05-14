# 【Cocos Creator】切换状态立即响应组件触发 - 2
- pochiko 2019-05-13

依然是狗动画的切换
这次是成长值相关的切换，给定不同状态所需成长值，和实际成长值，来切换状态

原先是这样
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
        if(this._status !== s){
            this._status = s;
            updateStatus();
        }
    }

    updateStatus(){
        dogAni.play(`dog_${this._status}`);
    }
}
```

那么就需要定义一个总成长值和当前成长值
```typescript
    ... //代码的前面部分
    private _periodGrows: number[] = [];
    private _grow: number = 0;

    public set periodGrows(grows: number[]){
        this._periodGrows = grows;
    }

    public get periodGrows(){
        return this._periodGrows;
    }

    public set grow(grow: number){
        this._grow = grow;
    }

    public get grow(){
        return this._grow;
    }
    ... //代码的后面部分
```

定义好后，根据阶段成长和当前成长的setter来修改状态
```typescript
import _ from 'lodash'; //本代码使用了lodash的_.sortLastIndex(); 通过node去安装

    ... //代码的前面部分
    private _periodGrows: number[] = [];
    private _grow: number = 0;

    public set periodGrows(grows: number[]){
        this._periodGrows = grows;
        updateGrow();
    }

    public get periodGrows(){
        return this._periodGrows;
    }

    public set grow(grow: number){
        this._grow = grow;
        updateGrow();
    }

    public get grow(){
        return this._grow;
    }

    private updateGrow(){
        let periodGrows = this._periodGrows;
        let grow = this._grow;

        let status = _.sortedLastIndex(periodGrows, grow) - 1;  //比如，在[0, 15, 40]的成长值中，希望0在0状态中，15在1状态中，16在1状态中
        this.status = status;
    }
    ... //代码的后面部分
```

### 注：如果不使用lodash的方式，也可以自行写一个(此处要求arr是已经升序的数组)入库，然后从该库中调用
```typescript
export function sortedLastIndex<T extends number>(arr: T[], current: T){
    for(let i = 0; i <= arr.length; i++){
        //不是已经升序的数组，返回-1，其实也可以不使用这个判断。
        if(i > 0 && arr[i] > arr[i + 1]){
            return -1;  
        }
        if(current < arr[i]){
            return i;
        }
    }
}
```

[返回主页](./readme.md)
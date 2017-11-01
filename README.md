# piano

> intelligent piano


## Description
Web钢琴键盘，支持音程，和弦，调式等的动态计算；基于其精确到音符的数据组织方式，得以实现对音阶的完全掌控。

* 自由换肤, 动态渲染前端UI
* 智能的调式音阶计算
* 自由切换键盘键数
* 实时的琴键震动频率展示,方便乐器定音、调音
* 自动计算根音，主因
* 自动检测和弦类型；智能和弦构建

You can view the demo at <https://cuiyongjian.github.io/piano/>


## 音阶数据结构
``` javascript
{
  group: 0, // 组别(国际谱)
  innerIndex: 0, // 组内12平均索引号
  innerName: 'C', // 组内音名
  name: getter () {return this.innerName + this.innerIndex}
}
```

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build
```

For detailed explanation on how things work, consult the [docs for vue-loader](http://vuejs.github.io/vue-loader).

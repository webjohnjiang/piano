<template>
  <!-- <div id="app">
    <img src="./assets/logo.png">
    <h1>{{ msg }}</h1>
    <h2>Essential Links</h2>
    <ul>
      <li><a href="https://vuejs.org" target="_blank">Core Docs</a></li>
      <li><a href="https://forum.vuejs.org" target="_blank">Forum</a></li>
      <li><a href="https://gitter.im/vuejs/vue" target="_blank">Gitter Chat</a></li>
      <li><a href="https://twitter.com/vuejs" target="_blank">Twitter</a></li>
    </ul>
    <h2>Ecosystem</h2>
    <ul>
      <li><a href="http://router.vuejs.org/" target="_blank">vue-router</a></li>
      <li><a href="http://vuex.vuejs.org/" target="_blank">vuex</a></li>
      <li><a href="http://vue-loader.vuejs.org/" target="_blank">vue-loader</a></li>
      <li><a href="https://github.com/vuejs/awesome-vue" target="_blank">awesome-vue</a></li>
    </ul>
  </div> -->

<div id="app">
  <div>
    <h2>键盘配置</h2>
    <label for="isShowRecordKey">是否高亮按键</label>
    <select id="isShowRecordKey" v-model="isShowRecordKey">
      <option :value="true">记录</option>
      <option :value="false">不记录</option>
    </select>
  </div>
  <ul class="piano-panel">
    <template v-for="(pianoKey, index) of pianoKeys">
      <li v-if="pianoKey.name.indexOf('#') > -1" v-bind:key="index" class="black-key" :title="`频率值: ${pianoKey.rate.toFixed(2)}`">
        <span @click.stop.prevent="selectKey(pianoKey, $event)" :class="{active: isShowRecordKey && (selectedKeys.indexOf(pianoKey) > -1), current: selectedKeys[selectedKeys.length - 1] === pianoKey}"><i>{{pianoKey.name}}</i></span>
      </li>
      <li v-if="pianoKey.name.indexOf('#') == -1" v-bind:key="index" class="white-key" :title="`频率值: ${pianoKey.rate.toFixed(2)}`" @click.stop.prevent="selectKey(pianoKey, $event)" :class="{active: isShowRecordKey && (selectedKeys.indexOf(pianoKey) > -1), current: selectedKeys[selectedKeys.length - 1] === pianoKey}">
        <span><i>{{pianoKey.name}}</i></span>
      </li>
    </template>
  </ul>
  <!-- <div class="current-key" v-if="selectedKeys && selectedKeys.length">
    {{JSON.stringify(selectedKeys[selectedKeys.length - 1])}}
  </div> -->
  <div class="key-distance">
    <h2>
      音程: 
      <span v-if="distance">
        <span v-html="distance"></span>
        ({{`${selectedKeys[selectedKeys.length - 2].name} - ${selectedKeys[selectedKeys.length - 1].name}`}})
      </span>
      <span v-else>_</span>
    </h2>
  </div>
  <div class="key-chord">
    <h2>和弦类型: {{chord}}</h2>
    <h2>根音: {{rootKey}}</h2>
    <!-- <h2>低音: {{}}</h2> -->
  </div>
</div>
</template>

<script>

import Vue from 'vue'

class fingerStore {
  constructor() {
    this._store = []
  }
  add(key) {
    if (this._store.length > 10) {
      this._store.shift()
    }
    this._store.push(key)
  }
  get(index) {
    return this.store[index]
  }
  last() {
    return this.store[this.store.length - 1]
  }
  length() {
    return this._store.length
  }
  slice(start, end) {
    return this._store.slice(start, end)
  }
}

export default {
  name: 'app',
  data () {
    return {
      isShowRecordKey: true,
      _pianoKeys: null, // 全部的原始音阶信息
      // selectedKeys: new fingerStore() // 最近按下的10个按键(手指只有10个)
      selectedKeys: [], // 最近按下的10个按键
    }
  },
  computed: {
    pianoKeys () {
      if (!this._pianoKeys) {
        return []
      }
      // 用于UI渲染的钢琴键盘结构
      let start = this._pianoKeys.findIndex(pkey=>pkey.name==='A0')
      let end = this._pianoKeys.findIndex(pkey=>pkey.name==='C8') + 1
      return this._pianoKeys.slice(start, end)
    },
    distance () {
      // 求最近2个按键的音程，只需求出在原始12平均律键盘排列this._piano中的间距
      // 由于每个key里包含了group和innerIndex，所以可以算出其原始下标信息，所以可以直接减。
      // 方法2: 根据音名做算法(该方法不太基于本质)。
      if (this.selectedKeys.length >= 2) {
        let keyOne = this.selectedKeys[this.selectedKeys.length - 2]
        let keyTwo = this.selectedKeys[this.selectedKeys.length - 1]
        let keyCounts = (keyTwo.group - keyOne.group) * 12 + keyTwo.innerIndex - keyOne.innerIndex
        let loopCounts = Math.abs(parseInt(keyCounts / 12)) // 八度的个数
        keyCounts = keyCounts % 12
        // 度数名字与keyCounts的关系，请移步: https://bideyuanli.com/p/3673
        let distanceNames = ['纯一度', '小二度', '大二度', '小三度', '大三度', '纯四度', '减五度', '纯五度', '小六度', '大六度', '小七度', '大七度', '纯八度']
        let distanceName = distanceNames[Math.abs(keyCounts)] // 音程名
        console.log('音程', loopCounts, distanceName, keyCounts)
        let relStr = (loopCounts ? (`<span class="distance-name">${loopCounts}个八度</span>, 另加一个`) : ('')) + `<span class="distance-name">${distanceName}</span>`
        return relStr
      }
      else {
        return null
      }
    },
    chord () {
      return null
    },
    rootKey () {
      return null
    }
  },
  watch: {
    isShowRecordKey () {
      console.log('isShowRecordKey', this.isShowRecordKey)
      // 切换模式时，清空已记录的琴键
      this.selectedKeys = []
    }
  },
  methods: {
    selectKey (key, event) {
      console.log('judge active1: ', this.isShowRecordKey && (this.selectedKeys.indexOf(key) > -1))
      let findKeyIndex = this.selectedKeys.indexOf(key)
      if (this.isShowRecordKey && findKeyIndex > -1) {
        // 记录模式，两次点击同一个按键，要取消掉记录
        this.selectedKeys.splice(findKeyIndex, 1)
      }
      else {
        if (this.selectedKeys.length > 10) {
          this.selectedKeys.shift()
        }
        this.selectedKeys.push(key)
      }
      
      console.log('judge active2: ', ((this.isShowRecordKey) && (this.selectedKeys.indexOf(key) > -1)))
      console.log(this.isShowRecordKey)
      console.log(this.selectedKeys.indexOf(key) > -1)
      // if (this.isShowRecordKey) {
      //   // 若要键盘上记录按过的key，则点击后添加按键的active class
      //   key.active = !key.active
      //   console.log(key)
      //   // console.log(event)
      //   // event.currentTarget.classList.add('active')
      // }
    }
  },
  // 使用算法生成88个音阶
  created () {
    // var keyNames = ['C', 'D', 'E', 'F', 'G', 'A', 'B']
    // var firstKey = {
    //   name: 'A',
    //   group: 0,
    //   rate: 27.5,
    // }
    // var fullKeys = [firstKey]
    // // 按照全全依次推算出后87个音级
    // while(true) {
    //   let lastKey = fullKeys[fullKeys.length - 1]
    //   if (lastKey.name != 'C')
    // }
  
    
    // 生成9组全键盘
    var fullKeys = []
    for (let i = 0; i < 9; i++) {
      for (let j = 0; j < 12; j++) {
        let keyNames = ['C', '#C', 'D', '#D', 'E', 'F', '#F', 'G', '#G', 'A', '#A', 'B']
        let keyInfo = {
          group: i, // 组别(国际谱)
          innerIndex: j, // 组内12平均索引号
          innerName: keyNames[j], // 组内音名
          active: false
        }
        Object.defineProperty(keyInfo, 'name', {
          get: function () {
            return this.innerName + this.group
          },
          enumerable: true
        })
        fullKeys.push(keyInfo)
      }
    }

    // // 按照C-A的音名顺序，以及全全半全全全半的规定，标记上组内音名
    // let rule = [1, 1, 0, 1, 1, 1, 0]
    // fullKeys.forEach((item, index) => {

    // })

    // 基于中央C给每个键添加频率值
    let centerC = fullKeys.findIndex(item=>{return item.name === 'A4'})
    console.log('centerC is ', centerC)
    fullKeys[centerC].rate = 440
    // 对2开12次方根，求出12平均律中这个频率等比数列的比数
    let rateStep = Math.pow(2, 1/12)
    console.log(rateStep)
    for (let i = centerC + 1; i < fullKeys.length; i++) {
      fullKeys[i].rate = fullKeys[i - 1].rate * rateStep
    }
    for (let i = centerC - 1; i >= 0; i--) {
      fullKeys[i].rate = fullKeys[i + 1].rate / rateStep
    }
    this._pianoKeys = fullKeys
  },
  mounted () {
    console.log('ready add click handler')
    document.addEventListener('click', (e)=>{
      console.log('what', e.target)
      if (this.isShowRecordKey) {
        // let activeKeys = document.querySelectorAll('.piano-panel li.active, .piano-panel li span.active')
        // activeKeys.forEach(item=>{
        //   item.classList.remove('active')
        //   item.classList.remove('current')
        // })
        // 清空selectedKeys就可以了
        this.selectedKeys = []
      }
    }, false)
  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

h1, h2 {
  font-weight: normal;
}

ul.piano-panel {
  list-style-type: none;
  padding: 0;
  display: flex;
  position: relative;
}

.piano-panel li.white-key {
  margin: 0px;
  background: #fff;
  padding: 40px 2px 10px;
  border: 1px solid #a9a9a9;
  /* overflow: hidden; */
  width: 20px;
  font-size: 12px;
  flex: 1;
  min-width: 20px;
  max-width: 20px;
  cursor: pointer;
}
.piano-panel li.white-key:hover, li.white-key.active {
  background: #dedede;
}

.piano-panel li.black-key {
  width: 0;
  position: relative;
  cursor: pointer;
}
.piano-panel li.black-key span {
  background: black;
  position: absolute;
  width: 12px;
  min-width: 12px;
  max-width: 12px;
  left: -6px;
  display: inline-block;
  padding: 10px 1px;
  color: white;
  font-size: 12px;
}
.piano-panel li.black-key span i {
  font-size: 12px;
  transform: scale(0.5, 0.5);
  display: inline-block;
  width: 100%;
  height: 100%;
  /* position: absolute; */
  transform: translateX(-3px) scale(0.6,0.6);
}

.piano-panel li.black-key span:hover, li.black-key span.active {
  background: #828282;
}

.piano-panel li.white-key.current {
  background: #e3f37f
}
.piano-panel li.black-key span.current {
  background: #e3f37f;
  color: black;
}

.key-distance .distance-name {
  color: #849a00;
}
</style>

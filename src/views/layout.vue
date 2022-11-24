<template>
  <div class="game">
    <!-- 游戏画面 -->
    <div class="game-screen">
      <!-- 主屏幕 -->
      <div class="home-screen">
        <!-- 一行 -->
        <div class="row" v-for="(row, i) in frame" :key="i">
          <div
            class="min-block"
            :style="{ background: block.bg }"
            v-for="(block, j) in row"
            :key="j"
          ></div>
        </div>
      </div>
      <!-- 副屏幕 -->
      <div class="subscreen">
        <div class="top" style="text-align: center">
          <span style="color: rgba(238, 238, 238, 0.822); line-height: 25px"
            >NEXT</span
          >
          <div class="row" v-for="(row, i) in subscreen" :key="i">
            <div
              class="min-block"
              :style="{ background: block.bg }"
              v-for="(block, j) in row"
              :key="j"
            ></div>
          </div>
        </div>
        <div class="item">
          <span>得分</span>
          <div class="box">0</div>
        </div>
        <div class="item">
          <span>等级</span>
          <div class="box">1</div>
        </div>
        <div class="item">
          <span>消除</span>
          <div class="box">0</div>
        </div>
        <div class="switch">
          <button>暂停</button>
          <button>开始</button>
        </div>
      </div>
    </div>
    <!-- 操作区域 -->
    <div class="operation-area">
      <div class="control">
        <button>变化</button>
        <button>向下</button>
        <button>向左</button>
        <button>向右</button>
      </div>
    </div>
  </div>
</template>

<script>
import { blockMod, color, transition } from '@/utils/data'
export default {
  data () {
    return {
      row: 20,
      col: 10,
      frame: [], // 主屏幕
      subscreen: [], // 副屏幕
      bg: '#142957',
      block: [], // 方块集和
      now: { b: 0, c: 0 }, // 当前方块以及旋转角度
      next: { b: 0, c: 0 }, // 下一个方块以及旋转角度
      nowBlock: {}, // 当前方块的形状数据
      nextBlock: {} // 下一个方块的形状数据
    }
  },
  methods: {
    // 游戏框架
    gameFrame () {
      for (let i = 0; i < this.row; i++) {
        const a = []
        for (let j = 0; j < this.col; j++) {
          const b = {
            data: 0,
            bg: this.bg
          }
          a.push(b)
        }
        this.frame.push(a)
      }
      for (let i = 0; i < 4; i++) {
        const a = []
        for (let j = 0; j < 4; j++) {
          const b = {
            data: 0,
            bg: this.bg
          }
          a.push(b)
        }
        this.subscreen.push(a)
      }
    },
    // 获取渲染方块
    getBlock (index) {
      this.block = blockMod(color[index])
    },
    // 渲染下一个形状
    async getNext () {
      // 随机形状
      this.next.b = Math.floor(Math.random() * this.block.length)
      // 获取旋转方式
      this.next.c = Math.floor(Math.random() * 4)
    },
    // 渲染当前的形状（当前形状和下一个形状相同）
    init () {
      // 获取到下一个形状数据
      this.now = JSON.parse(JSON.stringify(this.next))
      // 当前的形状数据
      this.nowBlock = JSON.parse(JSON.stringify(this.block[this.now.b]))
      this.renderBlock(this.nowBlock, this.frame, 1)
      // 旋转
      if (this.now.c > 0) {
        for (let i = 0; i < this.now.c; i++) {
          this.change(this.nowBlock, this.frame, this.now, i)
        }
      }
      this.getNext().then(() => {
        this.nextBlock = JSON.parse(JSON.stringify(this.block[this.next.b]))
        this.renderBlock(this.nextBlock, this.subscreen, 1)
        // 旋转
        if (this.next.c > 0) {
          for (let i = 0; i < this.next.c; i++) {
            this.change(this.nextBlock, this.subscreen, this.next, i)
          }
        }
      })
    },
    // 渲染形状 b是方块 d是位置  n：0 是擦除 1是生成 2确定落到最下层
    renderBlock (b, d, n) {
      const c = b.site
      if (n === 0) {
        for (let i = 0; i < c.length; i += 2) {
          d[c[i]][c[i + 1]].bg = this.bg
        }
      } else if (n === 1) {
        for (let i = 0; i < c.length; i += 2) {
          d[c[i]][c[i + 1]].bg = b.color
        }
      } else if (n === 2) {
        for (let i = 0; i < c.length; i += 2) {
          d[c[i]][c[i + 1]].data = 1
        }
      }
    },
    // b: 当前方块，d: 要渲染的位置，z：渲染的对象现在还是下一个，xz:当前旋转角度
    change (b, d, z, xz) {
      // 先清空
      this.renderBlock(b, d, 0)
      // 记录方块第一块的位置
      const x = b.site[0]
      const y = b.site[1]
      for (let i = 0; i < b.site.length; i += 2) {
        const a = b.site[i]
        b.site[i] = b.site[i + 1] - y + x + transition[z.b][xz].x
        b.site[i + 1] = -(a - x) + y + transition[z.b][xz].y
      }
      xz++
      if (xz === 4) {
        xz = 0
      }
      this.renderBlock(b, d, 1)
    }
  },
  mounted () {
    this.gameFrame()
    this.getBlock(0)
    this.getNext()
    this.init()
  }
}
</script>

<style lang="less" scoped>
.game {
  display: flex;
  flex-direction: column;
  height: 100vh;
  border: 8px solid skyblue;
  .game-screen {
    padding: 10px;
    display: flex;
    .home-screen {
      margin-right: 15px;
    }
    .subscreen {
      display: flex;
      flex-direction: column;
      align-items: center;
      flex: 1;
      background: url(../assets/rect.png) no-repeat 0 0 / cover;
      .item {
        margin-top: 5px;
        width: 84px;
        text-align: center;
        span {
          color: rgba(238, 238, 238, 0.822);
        }
        .box {
          margin-top: 5px;
          padding: 5px 0;
          border: 1px solid #eee;
          border-radius: 15px;
          color: yellow;
          background-color: rgb(198 163 163 / 36%);
        }
      }
      .switch {
        margin-top: 15px;
        button {
          border: 0;
          padding: 0;
          width: 54px;
          height: 28px;
          border-radius: 14px;
          &:nth-child(1) {
            margin-right: 2px;
            background-color: #1989fa;
          }
          &:nth-child(2) {
            background-color: #07c160;
          }
        }
      }
    }
  }
  .operation-area {
    flex: 1;
    padding: 10px;
    background-color: rgba(255, 255, 255, 0.05);
    .control {
      position: relative;
      button {
        border: 0;
        width: 100px;
        height: 38px;
        border-radius: 19px;
        color: #fff;
        font-weight: 500;
        &:nth-child(1) {
          position: absolute;
          top: 35px;
          left: 120px;
          background-color: #006cff;
        }
        &:nth-child(2) {
          position: absolute;
          top: 95px;
          left: 120px;
          background-color: #32c5e9;
        }
        &:nth-child(3) {
          position: absolute;
          top: 65px;
          left: 10px;
          background-color: #60cda0;
        }
        &:nth-child(4) {
          position: absolute;
          top: 65px;
          left: 228px;
          background-color: #00eff6;
        }
      }
    }
  }
}
.row {
  display: flex;
  margin-bottom: 1px;
  .min-block {
    margin-right: 1px;
    width: 20px;
    height: 20px;
  }
}
</style>

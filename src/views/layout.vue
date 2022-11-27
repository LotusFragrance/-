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
          <div class="box">{{ score }}</div>
        </div>
        <div class="item">
          <span>等级</span>
          <div class="box grade">
            <button @click="downFn">-</button>
            {{grade}}
            <button @click="upFn">+</button>
          </div>
        </div>
        <div class="item">
          <span>消除</span>
          <div class="box">{{removeRow}}</div>
        </div>
        <div class="switch">
          <button @click="stopGame">暂停</button>
          <button @click="startGame">开始</button>
        </div>
      </div>
    </div>
    <!-- 操作区域 -->
    <div class="operation-area">
      <div class="control">
        <button @click="change1">变化</button>
        <button @click="moveDown">向下</button>
        <button @click="moveLeft">向左</button>
        <button @click="moveRight">向右</button>
      </div>
      <div class="fix">
        创作者:LotusFragrance
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
      nextBlock: {}, // 下一个方块的形状数据
      xz: 0, // 旋转的角度
      timer: null, // 自动向下
      removeRow: 0, // 等级
      score: 0,
      removeAllBlock: [],
      grade: 1
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
      // 模拟格子被占用
      // this.frame[4][4].bg = 'skyblue'
      // this.frame[4][4].data = 1
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
      this.xz = this.now.c
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
        // 先擦除后渲染
        this.renderBlock(this.nowBlock, this.subscreen, 0)
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
    },
    // 下落后的旋转
    change1 () {
      const b = JSON.parse(JSON.stringify(this.nowBlock))
      // 记录方块第一块的位置
      const x = b.site[0]
      const y = b.site[1]
      let n = true
      for (let i = 0; i < b.site.length; i += 2) {
        const a = b.site[i]
        b.site[i] = b.site[i + 1] - y + x + transition[this.now.b][this.xz].x
        b.site[i + 1] = -(a - x) + y + transition[this.now.b][this.xz].y

        // 判断旋转后该点是否合理
        if (
          b.site[i + 1] < 0 ||
          b.site[i + 1] > this.col - 1 ||
          b.site[i] > this.row - 1 ||
          this.frame[b.site[i]][b.site[i + 1]].data === 1
        ) {
          n = false
        }
      }
      // 符合条件
      if (n) {
        this.renderBlock(this.nowBlock, this.frame, 0)
        this.xz++
        if (this.xz === 4) {
          this.xz = 0
        }
        this.nowBlock = b
        this.renderBlock(this.nowBlock, this.frame, 1)
      }
    },
    // 向下移动
    moveDown () {
      if (this.isMove(3)) {
        // 先清理在渲染形状数据
        this.renderBlock(this.nowBlock, this.frame, 0)
        for (let i = 0; i < this.nowBlock.site.length; i += 2) {
          // 向下移动一位
          this.nowBlock.site[i]++
        }
        this.renderBlock(this.nowBlock, this.frame, 1)
      } else {
        // 已经不能下落了
        this.renderBlock(this.nowBlock, this.frame, 2)
        this.isRemove()
      }
    },
    // 自动向下移动
    autMoveDown () {
      const speed = {
        1: 1000,
        2: 500,
        3: 300,
        4: 200,
        5: 100
      }
      this.timer = setInterval(() => {
        this.moveDown()
      }, speed[this.grade])
    },
    // 向左移动
    moveLeft () {
      if (!this.isMove(2)) return
      // 先清理在渲染形状数据
      this.renderBlock(this.nowBlock, this.frame, 0)
      for (let i = 0; i < this.nowBlock.site.length; i += 2) {
        // 向下移动一位
        this.nowBlock.site[i + 1]--
      }
      this.renderBlock(this.nowBlock, this.frame, 1)
    },
    // 向右移动
    moveRight () {
      if (!this.isMove(1)) return
      // 先清理在渲染形状数据
      this.renderBlock(this.nowBlock, this.frame, 0)
      for (let i = 0; i < this.nowBlock.site.length; i += 2) {
        // 向下移动一位
        this.nowBlock.site[i + 1]++
      }
      this.renderBlock(this.nowBlock, this.frame, 1)
    },
    // 判断是否可以移动
    // 参数e  1: 右移 2：左移 3： 下移 4： 变化
    isMove (e) {
      let d = 0
      // 获取坐标
      const c = this.nowBlock.site
      switch (e) {
        case 1:
          for (let i = 0; i < this.nowBlock.site.length; i += 2) {
            if (c[i + 1] >= this.col - 1) {
              return false
            }
            d += this.frame[c[i]][c[i + 1] + 1].data
          }
          if (d > 0) {
            return false
          }
          break
        case 2:
          for (let i = 0; i < this.nowBlock.site.length; i += 2) {
            if (c[i + 1] <= 0) {
              return false
            }
            d += this.frame[c[i]][c[i + 1] - 1].data
          }
          if (d > 0) {
            return false
          }
          break
        case 3:
          for (let i = 0; i < this.nowBlock.site.length; i += 2) {
            if (c[i] >= this.row - 1) {
              return false
            }
            // 判断下一个位置是否被占用
            d += this.frame[c[i] + 1][c[i + 1]].data
          }
          if (d > 0) {
            return false
          }
          break
      }
      return true
    },
    // 开始游戏
    startGame () {
      clearInterval(this.timer)
      this.autMoveDown()
    },
    // 暂停游戏
    stopGame () {
      clearInterval(this.timer)
    },
    // 判断是否可以消除
    isRemove () {
      for (let i = 0; i < this.row; i++) {
        let sum = 0
        for (let j = 0; j < this.col; j++) {
          sum += this.frame[i][j].data
        }
        if (sum === this.col) {
          // 获取可以消除的行
          this.removeAllBlock.push(i)
        }
      }
      // 判断是否可以消除行
      if (this.removeAllBlock.length > 0) {
        const arr = JSON.parse(JSON.stringify(this.removeAllBlock))
        this.removeAllBlock = []
        const p = new Promise((resolve, reject) => {
          for (let i = 0; i < arr.length; i++) {
            let j = 0
            // 让消除方块依次消除（200毫秒）
            const timer = setInterval(() => {
              this.frame[arr[i]][j] = {
                data: 0,
                bg: this.bg
              }
              j++
              if (j === this.col) {
                clearInterval(timer)
                if (i === arr.length - 1) {
                  resolve()
                }
              }
            }, 20)
          }
        })
        p.then(() => {
          // 得分 一行100, 两行300, 三行：600, 四行900...
          this.score += 100 * ((arr.length + 1) / 2) * arr.length
          // 消除行数
          this.removeRow += arr.length
          // 消除后方块下移
          // let newFrame = []
          // console.log(this.frame)
          // newFrame = this.frame.filter(item => {
          //   let isOk = false
          //   for (let i = 0; i < item.length; i++) {
          //     if (item[i].data === 1) {
          //       isOk = true
          //       continue
          //     }
          //   }
          //   return isOk
          // })
          // console.log(newFrame)
          // if (newFrame.length > 0) {
          //   // 先清除后渲染
          //   // this.delete()
          //   let j = 1
          //   for (let i = newFrame.length - 1; i >= 0; i--) {
          //     this.frame[this.row - j] = newFrame[i]
          //     // this.$set(this.frame, this.row - j, newFrame[i])
          //     this.delete(newFrame.length)
          //     j++
          //   }
          // }
          for (let i = this.row - 1; i >= 0; i--) {
            let a = 0
            for (let j = 0; j < arr.length; j++) {
              if (arr[j] > i) {
                a++
              }
            }
            if (a > 0) {
              for (let k = 0; k < this.col; k++) {
                if (this.frame[i][k].data === 1) {
                  // 先向下移动
                  this.frame[i + a][k] = this.frame[i][k]
                  // 再清楚当前
                  this.frame[i][k] = { data: 0, bg: this.bg }
                }
              }
            }
          }
          // 生成下一个
          this.init()
        })
      } else {
        this.init()
      }
    },
    downFn () {
      if (this.grade === 1) return false
      this.grade -= 1
    },
    upFn () {
      if (this.grade === 5) return false
      this.grade += 1
    },
    // 清除所有方块
    delete (length) {
      for (let i = 0; i < this.row - length; i++) {
        for (let j = 0; j < this.col; j++) {
          this.frame[i][j] = {
            data: 0,
            bg: this.bg
          }
        }
      }
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
  .grade {
    button {
      width: 27px;
      font-size: 16px;
      border-radius: 5px;
      background-color: #9d977e7a;
      color: rgba(238, 238, 238, 0.822);
    }
  }
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
    position: relative;
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
    .fix {
      position: absolute;
      bottom: 10px;
      right: 3px;
      color: #fff;
      opacity: .5;
      font-size: 12px;
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

<template>
  <div class="top">Canvas 案例 ———— 打砖块</div>
  
  <div class="break-ball" >
    <canvas id="breakBall" width="900" height="600"></canvas>
    
    <div class="container" v-if="show">
      <div class="dialog" v-if="start_game">
        <!-- <img src="" /> -->
        <div ></div>
        <div ></div>
        <div ></div>
        <p class="once-again">开始游戏</p>
        <el-button class="start" type="" @click="startGame" >开始</el-button>
      </div>
      <div class="dialog" v-if=" gameOver " >
        <p class="once-again">本轮分数：{{score}}分</p>
        <p class="once-again">真好玩！</p>
        <p class="once-again">再来一次~~</p>
        <el-button class="once-again-btn" type="" @click="init" >开始</el-button>
      </div>
    </div>
  </div>
</template>
  
<script>
const randomColor = [
  "#FF95CA",
  "#00E3E3",
  "#00E3E3",
  "#6F00D2",
  "#6F00D2",
  "#C2C287",
  "#ECFFFF",
  "#FFDC35",
  "#93FF93",
  "#d0d0d0"
];
export default {
  name: "BreakBall",
  data() {
    return {
      clientWidth: 0,
      clientHeight: 0,
      ctx: null,
      crushBalls: [],
      
      pannel: {
        //木板
        x: 0,
        y: 0,
        height: 8,
        width: 100,
        speed: 8,
        dx: 0
      },
      ball: {
        //小球
        x: 0,
        y: 0,
        r: 8,
        dx: -4,
        dy: -4
      },
      score: 0,
      gameOver: false,
      start_game: true,
      show:true,
      breaks: [],
      breaksConfig: {
        row: 6, // 排数
        height: 25, // 砖块高度
        width: 130, // 砖块宽度
        radius: 5, // 矩形圆角
        space: 0, // 间距
        colunm: 6 // 列数
      }
    };
  },
  mounted() {
    let _this = this;
    let container = document.getElementById("breakBall");
    this.ctx = container.getContext("2d");
    this.clientHeight = container.height;
    this.clientWidth = container.width;
    _this.init();
    document.onkeydown = function(e) {
      let key = window.event.keyCode;
      if (key === 37) {
        // 左键
        _this.pannel.dx = -_this.pannel.speed;
      } else if (key === 39) {
        // 右键
        _this.pannel.dx = _this.pannel.speed;
      }
      if(key === 13 )
      {
        //回车
        _this.init();
      }
    };
    
    document.onkeyup = function(e) {
      _this.pannel.dx = 0;
    };

    (function animloop() {//闭包
      if (!_this.gameOver && !_this.start_game) {
        _this.movePannel();
        _this.moveBall();
        _this.drawAll();
      } else {
        _this.drawCrushBreaks();

      }
      window.requestAnimationFrame(animloop);
    })();
  },
  computed:{
    showBreaksCount(){
      return this.breaks.filter(item=>{
        return item.show;
      }).length;
    }
  },
  methods: {

    startGame(){
      this.start_game = false
    },
    init() {
      let _this = this;
      _this.gameOver = false;

      this.pannel.y = this.clientHeight - this.pannel.height;
      this.pannel.x = this.clientWidth / 2 - this.pannel.width / 2;
      this.ball.y = this.clientHeight / 2;
      this.ball.x = this.clientWidth / 2;
      this.score = 0;
      this.ball.dx = [-1,1][Math.floor(Math.random() * 2)]*4;
      this.ball.dy = [-1,1][Math.floor(Math.random() * 2)]*4;
      this.crushBalls = [];
      this.breaks = [];
      // 计算得出砖块缝隙宽度
      this.breaksConfig.space = Math.floor(
        (this.clientWidth -
          this.breaksConfig.width * this.breaksConfig.colunm) /
          (this.breaksConfig.colunm + 1)
      );
      //得到每个砖块在画布中的x,y坐标（指的砖块左上角的坐标）
      for (let i = 0; i < _this.breaksConfig.row; i++) {
        for (let j = 0; j < _this.breaksConfig.colunm; j++) {
          _this.breaks.push({
            x: this.breaksConfig.space * (j + 1) + this.breaksConfig.width * j,
            y: 10 * (i + 1) + this.breaksConfig.height * i,
            show: true
          });
        }
      }
    },
    //调用所有动起来
    drawAll() {
      this.ctx.clearRect(0, 0, this.clientWidth, this.clientHeight);
      this.drawPannel();
      this.drawBall();
      this.drawScore();
      this.drawBreaks();
    },
    //移动木板
    movePannel() {
      this.pannel.x += this.pannel.dx;
      if (this.pannel.x > this.clientWidth - this.pannel.width) {
        this.pannel.x = this.clientWidth - this.pannel.width;
      } else if (this.pannel.x < 0) {
        this.pannel.x = 0;
      }
    },
    //小球的运动
    moveBall() {
      this.ball.x += this.ball.dx;
      this.ball.y += this.ball.dy;
      this.breaksHandle();
      this.edgeHandle();
    },
    breaksHandle() {
      // 触碰砖块检测
      this.breaks.forEach(item => {
        if (item.show) {
          if (
            this.ball.x + this.ball.r > item.x &&
            this.ball.x - this.ball.r < item.x + this.breaksConfig.width &&
            this.ball.y + this.ball.r > item.y &&
            this.ball.y - this.ball.r < item.y + this.breaksConfig.height
          ) {
            item.show = false;
            this.ball.dy *= -1;
            this.score ++ ;
            if(this.showBreaksCount === 0){
              this.gameOver = true;
              this.start_game = false;
              this.show = true
            }
          }
        }
      });
    },
    edgeHandle() {
      // 边缘检测
      // 碰到顶部反弹
      if (this.ball.y - this.ball.r < 0) {
        this.ball.dy = -this.ball.dy;
      }
      if (
        // 碰到左右墙壁
        this.ball.x - this.ball.r < 0 ||
        this.ball.x + this.ball.r > this.clientWidth
      ) {
        this.ball.dx = -this.ball.dx;
      }
      if (
        this.ball.x >= this.pannel.x &&
        this.ball.x <= this.pannel.x + this.pannel.width &&
        this.ball.y + this.ball.r >= this.clientHeight - this.pannel.height
      ) {
        // 球的x在板子范围内并触碰到了板子
        this.ball.dy *= -1;
      } else if (
        (this.ball.x < this.pannel.x ||
          this.ball.x > this.pannel.x + this.pannel.width) &&
        this.ball.y + this.ball.r >= this.clientHeight
      ) {
        // 球碰到了底边缘了
        this.gameOver = true;
        this.start_game = false;
        this.show = true;
        this.getCurshBreaks();
      }
    },
    //绘制分数
    drawScore(){
      this.ctx.beginPath();
      this.ctx.font="14px Arial";
      this.ctx.fillStyle = "#FFF";
      this.ctx.fillText("分数："+this.score,10,this.clientHeight-14);
      this.ctx.closePath();
    },
    drawCrushBreaks() {
      this.ctx.clearRect(0, 0, this.clientWidth, this.clientHeight);
      this.crushBalls.forEach(item => {
        this.ctx.beginPath();
        this.ctx.arc(item.x, item.y, item.r, 0, 2 * Math.PI);
        this.ctx.fillStyle = item.color;
        this.ctx.fill();
        this.ctx.closePath();
        item.x += item.dx;
        item.y += item.dy;
        if (
          // 碰到左右墙壁
          item.x - item.r < 0 ||
          item.x + item.r > this.clientWidth
        ) {
          item.dx = -item.dx;
        }
        if (
          // 碰到上下墙壁
          item.y - item.r < 0 ||
          item.y + item.r > this.clientHeight
        ) {
          item.dy = -item.dy;
        }
      });
    },
    getRandomColor() {
      return randomColor[Math.floor(Math.random() * randomColor.length)];
    },
    getRandomArbitrary(min, max) {
      return Math.random() * (max - min) + min;
    },
    //绘制结束动画
    getCurshBreaks() {
      let _this = this;
      this.breaks.forEach(item => {
        if (item.show) {
          item.show = false;
          for (let i = 0; i < 8; i++) {
            this.crushBalls.push({
              x: item.x + this.breaksConfig.width / 2,
              y: item.y + this.breaksConfig.height / 2,
              dx: _this.getRandomArbitrary(-6, 6),
              dy: _this.getRandomArbitrary(-6, 6),
              r: _this.getRandomArbitrary(1, 4),
              color: _this.getRandomColor()
            });
          }
        }
      });
    },
    //绘制小球
    drawBall() {
      this.ctx.beginPath();
      this.ctx.arc(this.ball.x, this.ball.y, this.ball.r, 0, 2 * Math.PI);
      this.ctx.fillStyle = "#008b8b";
      this.ctx.fill();
      this.ctx.closePath();
    },

    //绘制木板
    drawPannel() {
      this.drawRoundRect(
        this.pannel.x,
        this.pannel.y,
        this.pannel.width,
        this.pannel.height,
        5
      );
    },

    //画圆角矩形
    drawRoundRect(x, y, width, height, radius) {
      this.ctx.beginPath();
      this.ctx.arc(x + radius, y + radius, radius, Math.PI, (Math.PI * 3) / 2);
      this.ctx.lineTo(width - radius + x, y);
      this.ctx.arc(
        width - radius + x,
        radius + y,
        radius,
        (Math.PI * 3) / 2,
        Math.PI * 2
      );
      this.ctx.lineTo(width + x, height + y - radius);
      this.ctx.arc(
        width - radius + x,
        height - radius + y,
        radius,
        0,
        (Math.PI * 1) / 2
      );
      this.ctx.lineTo(radius + x, height + y);
      this.ctx.arc(
        radius + x,
        height - radius + y,
        radius,
        (Math.PI * 1) / 2,
        Math.PI
      );
      this.ctx.fillStyle = "#008b8b";
      this.ctx.fill();
      this.ctx.closePath();
    },
    //绘制砖块
    drawBreaks() {
      let _this = this;
      _this.breaks.forEach(item => {
        if (item.show) {
          _this.drawRoundRect(
            item.x,
            item.y,
            _this.breaksConfig.width,
            _this.breaksConfig.height,
            _this.breaksConfig.radius
          );
        }
      });
    }
  }
};
</script>
  
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.top{
  font-size: 28px;
}
.break-ball {
    width: 900;
    height: 900;
    display: flex;
    margin: auto;
    width: fit-content;
    position: relative;

  #breakBall {
    background: #2a4546;
  }
  .container {
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    background-color: rgba(0, 0, 0, 0.3);
    text-align: center;
    font-size: 0;
    white-space: nowrap;
    overflow: auto;
  }
  .container:after {
    content: "";
    display: inline-block;
    height: 100%;
    vertical-align: middle;
  }
  .dialog {
    width: 400px;
    height: 300px;
    background: rgba(255, 255, 255, 0.5);
    box-shadow: 3px 3px 6px 3px rgba(0, 0, 0, 0.3);
    display: inline-block;
    vertical-align: middle;
    text-align: left;
    font-size: 28px;
    color: #fff;
    font-weight: 600;
    border-radius: 10px;
    white-space: normal;
    text-align: center;
    .once-again-btn {
      background: #1f9a9a;
      border: none;
      color: #fff;
    }
  }
}
</style>



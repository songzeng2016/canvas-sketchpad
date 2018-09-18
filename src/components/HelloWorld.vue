<template>
  <div class="main">
    <div class="header">
      <button @click="startDrawing">截图</button>
      <button @click="drawOver">完成</button>
    </div>
    <div class="iframe-box">
      <iframe id="iframe" class="iframe" :src="url1" frameborder="0"></iframe>
      <div
        v-if="showMask"
        class="mask"
        ref="mask"
        @mousedown="startShot">
      </div>
      <div class="wrap" v-if="showShot">
        <div class="top" :style="topStyle"></div>
        <div class="left" :style="leftStyle"></div>
        <div class="box" :style="boxStyle">
          <div class="direction nw" @mousedown="changeShotSize($event, 'nw')"></div>
          <div class="direction n" @mousedown="changeShotSize($event, 'n')"></div>
          <div class="direction ne" @mousedown="changeShotSize($event, 'ne')"></div>
          <div class="direction e" @mousedown="changeShotSize($event, 'e')"></div>
          <div class="direction se" @mousedown="changeShotSize($event, 'se')"></div>
          <div class="direction s" @mousedown="changeShotSize($event, 's')"></div>
          <div class="direction sw" @mousedown="changeShotSize($event, 'sw')"></div>
          <div class="direction w" @mousedown="changeShotSize($event, 'w')"></div>
          <canvas class="canvas" id="canvas" :style="canvasStyle" @mousedown="canvasMove"></canvas>
          <div class="board" v-if="showBoard" :style="boardStyle">
            <ul class="tool">
              <li
                class="list"
                v-for="(item, index) in toolList"
                :key="item"
                :class="{active: index === toolIndex}"
                @click="selectTool(index)">
                <i class="iconfont" :class="['icon-' + item, item]"></i>
                <span v-if="index === toolList.length - 1" class="text">完成</span>
              </li>
            </ul>
            <div class="setting" v-if="showColor">
              <div class="left">
                <ul class="line">
                  <li class="list" :class="{active: lineSize === 0}" @click="lineSize = 0">
                    <div class="round round1"></div>
                  </li>
                  <li class="list" :class="{active: lineSize === 1}" @click="lineSize = 1">
                    <div class="round round2"></div>
                  </li>
                  <li class="list" :class="{active: lineSize === 2}" @click="lineSize = 2">
                    <div class="round round3"></div>
                  </li>
                </ul>
                <!--<div class="font"></div>-->
              </div>
              <div class="color">
                <div class="show" :style="{background: selectColor}"></div>
                <ul class="color-list">
                  <li
                    class="list"
                    v-for="item in colorList"
                    :key="item"
                    :style="{background: item}"
                    @click="selectColor = item">
                  </li>
                </ul>
              </div>
            </div>
          </div>
        </div>
        <div class="right" :style="rightStyle"></div>
        <div class="bottom" :style="bottomStyle"></div>
      </div>
    </div>
    <!--<canvas class="canvas"></canvas>-->
    <!--<button @click="screenshot">截图111</button>-->
    <!--<img :src="imgSrc" alt="">-->
  </div>
</template>

<script>
  import html2canvas from 'html2canvas';
  import axios from 'axios';
  import qs from 'qs';

  import Vue from 'vue';
  import bubble from 'components/bubble';

  const toolList = ['square', 'circular', 'arrow', 'brush', 'font', 'undo', 'close', 'check'];
  const colorList = ['#000000', '#ff0000', '#0dc8bd', '#b9b9b9', '#59c1eb', '#ebd135', '#ebac32', '#63d400',
    '#8fb91e', '#0aa5eb', '#706eaa', '#b9348e', '#559ab9', '#c3eb2d', '#e73eeb', '#601aeb'];

  export default {
    name: 'HelloWorld',
    data() {
      return {
        toolList,
        toolIndex: '',
        colorList,
        selectColor: colorList[0],
        lineSize: 1,
        showMask: false,
        showShot: false,
        showBoard: false,
        showColor: false,
        mask: {
          top: '',
          left: '',
          width: '',
          height: '',
          right: '',
          bottom: '',
        },
        boardStyle: {
          top: '',
          right: 0,
        },
        imgSrc: '',
        imgList: [],
        width: 0,
        height: 0,
        top: 0,
        left: 0,
        url1: `/static/pdfjs/web/viewer.html?file=/static/500.pdf`,
        url: `/static/pdfjs/web/viewer.html?file=http://192.168.1.228/file/2018-09-04/c24237dc-2fce-42a5-a888-c8c42d3e57b8_4589_1536042539661.pdf`,
        src: 'viewer.html/http://www.zhuyun365.com/file/2018-08-28/18087014-a2c8-4de2-8b1a-4ea75f7728f6_9246_1535438712777.pdf'
      };
    },
    created() {
      // axios.post('/api/user/login', qs.stringify({
      //   username: 'sz',
      //   password: '96e79218965eb72c92a549dd5a330112',
      //   clienttype: 'web',
      // })).then(json => {
      //
      // });
      window.renderBubble = this.renderBubble;
    },
    mounted() {

    },
    destroyed() {
      window.renderBubble = null;
    },
    computed: {
      // 计算大小与位置
      boxStyle() {
        return {
          width: this.width + 'px',
          height: this.height + 'px',
          top: this.top + 'px',
          left: this.left + 'px',
        };
      },
      topStyle() {
        return {
          height: this.top + 'px',
        };
      },
      leftStyle() {
        return {
          top: this.top + 'px',
          height: this.height + 'px',
          width: this.left + 'px',
        };
      },
      rightStyle() {
        return {
          left: this.left + this.width + 'px',
          top: this.top + 'px',
          height: this.height + 'px',
          width: (this.mask.width - this.left - this.width) + 'px',
        };
      },
      bottomStyle() {
        return {
          top: this.top + this.height + 'px',
          height: (this.mask.height - this.top - this.height) + 'px',
        };
      },
      canvasStyle() {
        return {
          cursor: this.showColor ? 'crosshair' : 'move',
        };
      },
      // 获取线条大小
      getLineSize() {
        let lineSize = 2;
        switch (this.lineSize) {
          case 0:
            lineSize = 2;
            break;
          case 1:
            lineSize = 4;
            break;
          case 2:
            lineSize = 8;
            break;
        }
        return lineSize;
      },
    },
    methods: {
      // 渲染气泡
      renderBubble(id) {
        let iframe;
        if (document.frames) {  // IE
          iframe = document.frames['iframe'];
        } else {
          iframe = document.getElementById('iframe').contentWindow;
        }

        let pageContainerNode = iframe.document.getElementById('pageContainer' + id);
        let bubbleLayerNode = pageContainerNode.getElementsByClassName('bubbleLayer')[0];

        let div = document.createElement('div');
        div.className = 'bubbleBox';
        // console.log(bubble);

        const template = `<div class="bubbleBox" @click="bubbleClick(id)"></div>`;
        const render = Vue.compile(template).render;

        bubbleLayerNode.appendChild(div);
      },
      // 设置选择面板位置
      setBoardPosition() {
        const boardWidth = 323;
        const boardHeight = 53;

        if (this.width < boardWidth && this.left < boardWidth - this.width) {
          this.boardStyle.right = `${this.left - (boardWidth - this.width)}px`;
        } else {
          this.boardStyle.right = '';
        }

        if (this.mask.height - this.height < boardHeight) {
          this.boardStyle.top = 0;
        } else if (this.top + this.height + boardHeight > this.mask.height) {
          this.boardStyle.top = `-${boardHeight}px`;
        } else {
          this.boardStyle.top = '';
        }
      },
      // 设置选择框显示
      setBoardShow() {
        const canvas = document.getElementById('canvas');
        canvas.width = this.width;
        canvas.height = this.height;
        this.showBoard = true;
        this.setBoardPosition();
      },
      // 完成截图
      drawOver() {
        const canvas = document.getElementById('canvas');
        let iframe;
        if (document.frames) {  // IE
          iframe = document.frames['iframe'];
        } else {
          iframe = document.getElementById('iframe').contentWindow;
        }

        let zoom = iframe.document.getElementById('scaleSelect').value;
        let topOffset = iframe.document.getElementById('viewerContainer').scrollTop;
        let leftOffset = iframe.document.getElementById('viewerContainer').scrollLeft;
        // let lang = iframe.currentLang;
        let getData = {
          x: this.left,
          y: this.top,
          w: this.width,
          h: this.height,
          width: parseInt(document.documentElement.clientWidth),
          height: parseInt(document.documentElement.clientHeight),
          path: '',
          scrollY: topOffset,
          scrollX: leftOffset,
          zoom: zoom,
          pageNo: parseInt(iframe.document.getElementById('pageNumber').value),
          timeout: '',
          imgStr: canvas.toDataURL(),
          location: 'http://192.168.1.228/file/2018-09-04/c24237dc-2fce-42a5-a888-c8c42d3e57b8_4589_1536042539661.pdf',
          url: 'http://www.czloud.com/screen/getshot/v1/',
          currentW: '',
          currentH: '',
        };
        console.log(getData);

        axios.post('api/extResource/getgraphic', qs.stringify(getData)).then(json => {

        });
      },
      // 开始截图
      startDrawing() {
        if (!this.showMask) {
          const headerHeight = 50;  // header 高度
          const borderSize = 4;  // mask 边框宽度

          this.showMask = true;
          this.$nextTick(() => {
            const mask = this.$refs.mask;
            let {offsetTop, offsetLeft, offsetWidth, offsetHeight} = mask;
            offsetTop = offsetTop + headerHeight;
            // 记录遮罩框的位置与大小
            this.mask = {
              top: offsetTop,
              left: offsetLeft,
              width: offsetWidth,
              height: offsetHeight,
              right: offsetLeft + offsetWidth - borderSize,
              bottom: offsetTop + offsetHeight - borderSize,
            };
          });
        }
      },
      // 生成图片
      screenshot() {
        html2canvas(document.getElementById('canvas')).then(canvas => {
          // document.body.appendChild(canvas);
          var canvasUrl = canvas.toDataURL();
          // console.log(canvasUrl);
          this.imgSrc = canvasUrl;
        });
      },
      // 画出截图框
      startShot(e) {
        // 记录开始位置
        const startX = e.x;
        const startY = e.y;
        // 设置截图框位置
        this.top = startY - this.mask.top;
        this.left = startX - this.mask.left;
        // 显示截图框
        this.showMask = false;
        this.showShot = true;

        // 记录初始坐标
        document.onmousemove = e => {
          // 记录移动位置
          let moveX = e.x;
          let moveY = e.y;
          // 判断移动位置，限制不能超出遮罩框
          if (moveY < this.mask.top) {
            moveY = this.mask.top;
          }
          if (moveY > this.mask.bottom) {
            moveY = this.mask.bottom;
          }
          if (moveX < this.mask.left) {
            moveX = this.mask.left;
          }
          if (moveX > this.mask.right) {
            moveX = this.mask.right;
          }

          // 设置截图框大小
          let width = moveX - startX;
          let height = moveY - startY;
          this.width = Math.abs(width);
          this.height = Math.abs(height);
          // 判断截图框位置
          // 预留 bug
          if (width < 0) {
            this.left = moveX - this.mask.left;
          }
          if (height < 0) {
            this.top = moveY - this.mask.top;
          }
        };
        document.onmouseup = () => {
          // 清空事件
          document.onmousemove = null;
          document.onmouseup = null;
          this.setBoardShow();
        };
      },
      // 改变截图框的大小
      changeShotSize(e, direction) {
        let startX = e.x;
        let startY = e.y;
        let startWidth = this.width;
        let startHeight = this.height;
        const startTop = this.top;
        const startLeft = this.left;

        // 定义拉伸函数
        const stretch = {
          top: (moveY) => {
            let height = moveY - startY;
            if (height < startHeight) {
              this.top = startTop + height;
            }
            // this.top = startTop + (height > startHeight ? startHeight : height);
            this.height = Math.abs(startHeight - height);
          },
          bottom: (moveY) => {
            let height = moveY - startY;
            if (-height > startHeight) {
              this.top = startTop + height + startHeight;
            }
            // this.top = startTop + (-height > startHeight ? height + startHeight : 0);
            this.height = Math.abs(startHeight + height);
          },
          left: (moveX) => {
            let width = moveX - startX;
            if (width < startWidth) {
              this.left = startLeft + width;
            }
            this.width = Math.abs(startWidth - width);
          },
          right: (moveX) => {
            let width = moveX - startX;
            if (-width > startWidth) {
              this.left = startLeft + width + startWidth;
            }
            this.width = Math.abs(startWidth + width);
          },
        };

        this.showBoard = false;  // 隐藏工具面板

        document.onmousemove = e => {
          let moveX = e.x;
          let moveY = e.y;

          // 判断移动位置，限制不能超出遮罩框
          if (moveY < this.mask.top) {
            moveY = this.mask.top;
          }
          if (moveY > this.mask.bottom) {
            moveY = this.mask.bottom;
          }
          if (moveX < this.mask.left) {
            moveX = this.mask.left;
          }
          if (moveX > this.mask.right) {
            moveX = this.mask.right;
          }

          // 判断拉伸方向
          switch (direction) {
            case 'n':
              stretch.top(moveY);
              break;
            case 's':
              stretch.bottom(moveY);
              break;
            case 'e':
              stretch.right(moveX);
              break;
            case 'w':
              stretch.left(moveX);
              break;
            case 'nw':
              stretch.top(moveY);
              stretch.left(moveX);
              break;
            case 'ne':
              stretch.top(moveY);
              stretch.right(moveX);
              break;
            case 'se':
              stretch.bottom(moveY);
              stretch.right(moveX);
              break;
            case 'sw':
              stretch.bottom(moveY);
              stretch.left(moveX);
              break;
          }
        };
        document.onmouseup = () => {
          // 清空事件
          document.onmousemove = null;
          document.onmouseup = null;
          this.setBoardShow();
        };
      },
      // 移动截图框
      canvasMove(e) {
        if (this.showColor) {
          this.canvasDraw(e);
          return;
        }
        const startX = e.x;
        const startY = e.y;
        const top = this.top;
        const left = this.left;

        // this.showBoard = false;

        // 记录初始坐标
        document.onmousemove = e => {
          let top1 = top + (e.y - startY);
          let left1 = left + e.x - startX;
          if (top1 < 0) {
            top1 = 0;
          }
          if (top1 > this.mask.bottom - this.mask.top - this.height) {
            top1 = this.mask.bottom - this.mask.top - this.height;
          }
          if (left1 < 0) {
            left1 = 0;
          }
          if (left1 > this.mask.right - this.mask.left - this.width) {
            left1 = this.mask.right - this.mask.left - this.width;
          }

          this.top = top1;
          this.left = left1;
          this.setBoardPosition();
        };
        document.onmouseup = () => {
          document.onmousemove = document.onmouseup = null;
          this.setBoardShow();
        };
      },
      // 选择画图工具
      selectTool(index) {
        if (index === 6) {  // 关闭
          this.showShot = false;
        }
        if (index === 5) {  // 撤销
          if (!this.imgList.length) {
            return;
          }
          const canvas = document.getElementById('canvas');
          let ctx = canvas.getContext('2d');

          this.imgList.pop();

          var img = new Image();
          img.src = this.imgList[this.imgList.length - 1];
          ctx.clearRect(0, 0, canvas.width, canvas.height);
          ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
          return;
        }
        if (this.toolIndex === index) {
          this.toolIndex = '';
          this.showColor = false;
        } else {
          this.toolIndex = index;
          this.showColor = true;
        }
      },
      // canvas 绘制，区别属性和方法
      canvasDraw(e) {
        const x = e.offsetX;
        const y = e.offsetY;
        const canvas = document.getElementById('canvas');
        let ctx = canvas.getContext('2d');
        let flag = false;

        // 加载图片
        const loadImage = () => {
          if (!this.imgList.length) {
            return;
          }
          var img = new Image();
          // img.src = this.imgSrc;
          img.src = this.imgList[this.imgList.length - 1];
          ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
        };
        // 画矩形
        const drawRect = (e, offsetX, offsetY) => {
          ctx.clearRect(0, 0, canvas.width, canvas.height);
          loadImage();
          ctx.beginPath();
          ctx.lineWidth = this.getLineSize;  // 设置线条宽度
          ctx.strokeStyle = this.selectColor;  // 设置边框颜色
          ctx.strokeRect(x, y, offsetX - x, offsetY - y);
        };
        // 画笔
        const drawPencil = (e, offsetX, offsetY) => {
          if (flag) {
            ctx.lineTo(offsetX, offsetY);
            ctx.stroke(); // 调用绘制方法
          } else {
            ctx.beginPath();
            ctx.moveTo(x, y);
            ctx.lineWidth = this.getLineSize;  // 设置线条宽度
            ctx.strokeStyle = this.selectColor;  // 设置边框颜色
            flag = true;
          }
        };
        // 画箭头
        const drawArrow1 = (ctx, fromX, fromY, toX, toY) => {
          let theta = this.getLineSize + 12;
          let headlen = this.getLineSize + 15;

          // 计算各角度和对应的P2,P3坐标
          let angle = Math.atan2(fromY - toY, fromX - toX) * 180 / Math.PI;
          let angle1 = (angle + theta) * Math.PI / 180;
          let angle2 = (angle - theta) * Math.PI / 180;
          let topX = headlen * Math.cos(angle1);
          let topY = headlen * Math.sin(angle1);
          let botX = headlen * Math.cos(angle2);
          let botY = headlen * Math.sin(angle2);

          headlen = headlen - 2;
          theta = theta - 10;

          let angle11 = (angle + theta) * Math.PI / 180;
          let angle21 = (angle - theta) * Math.PI / 180;
          let topX1 = headlen * Math.cos(angle11);
          let topY1 = headlen * Math.sin(angle11);
          let botX1 = headlen * Math.cos(angle21);
          let botY1 = headlen * Math.sin(angle21);

          ctx.clearRect(0, 0, canvas.width, canvas.height);
          loadImage();
          ctx.save();
          ctx.beginPath();

          let arrowX = fromX - topX;
          let arrowY = fromY - topY;
          ctx.moveTo(fromX, fromY);

          arrowX = toX + topX1;
          arrowY = toY + topY1;
          ctx.lineTo(arrowX, arrowY);

          arrowX = toX + topX;
          arrowY = toY + topY;
          ctx.lineTo(arrowX, arrowY);

          ctx.lineTo(toX, toY);

          arrowX = toX + botX;
          arrowY = toY + botY;
          ctx.lineTo(arrowX, arrowY);

          arrowX = toX + botX1;
          arrowY = toY + botY1;
          ctx.lineTo(arrowX, arrowY);

          ctx.lineTo(fromX, fromY);

          ctx.fillStyle = this.selectColor;
          ctx.fill();
          ctx.restore();
        };
        // 画椭圆
        const drawOval = (x, y, width, height) => {
          let k = (width / 0.75) / 2;
          let w = width / 2;
          let h = height / 2;

          ctx.clearRect(0, 0, canvas.width, canvas.height);
          loadImage();
          ctx.lineWidth = this.getLineSize;  // 设置线条宽度
          ctx.strokeStyle = this.selectColor;  // 设置边框颜色
          ctx.save();
          ctx.beginPath();
          ctx.moveTo(x, y - h);
          ctx.bezierCurveTo(x + k, y - h, x + k, y + h, x, y + h);
          ctx.bezierCurveTo(x - k, y + h, x - k, y - h, x, y - h);
          ctx.restore();
          ctx.stroke();
        };

        document.onmousemove = e => {
          // let {offsetX, offsetY} = e;
          let offsetX = e.x - this.mask.left - this.left;
          let offsetY = e.y - this.mask.top - this.top;
          // 限制移动坐标范围
          if (offsetX < 0) {
            offsetX = 0;
          }
          if (offsetX > this.width) {
            offsetX = this.width;
          }
          if (offsetY < 0) {
            offsetY = 0;
          }
          if (offsetY > this.height) {
            offsetY = this.height;
          }
          switch (this.toolIndex) {
            case 0:
              drawRect(e, offsetX, offsetY);
              break;
            case 1:
              drawOval(x + (offsetX - x) / 2, y + (offsetY - y) / 2, (offsetX - x), (offsetY - y));
              break;
            case 2:
              drawArrow1(ctx, x, y, offsetX, offsetY);
              break;
            case 3:
              drawPencil(e, offsetX, offsetY);
              break;
          }
        };
        document.onmouseup = e => {
          this.imgSrc = canvas.toDataURL();
          this.imgList.push(this.imgSrc);
          loadImage();
          document.onmousemove = document.onmouseup = null;
        };
      },
    }
  };
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus" type="text/stylus" scoped>
  ul
    margin: 0
    padding: 0
    li
      list-style: none

  .main
    display: flex
    flex-direction: column
    height: 100%
    overflow: hidden
    .header
      height: 50px
      background: #2c3e50
    .iframe-box
      flex: 1
      position: relative
      .iframe, .mask, .wrap
        width: 100%
        height: 100%
      .mask, .wrap
        position: absolute
        top: 0
        left: 0
        border: 2px solid #09c
        box-sizing: border-box
        overflow: hidden
      .mask
        background: rgba(0, 0, 0, 0.4)
        cursor: crosshair
      .wrap
        & > div
          position: absolute
          z-index: 3
          box-sizing: border-box
          background: #000000
          opacity: 0.4
        .top, .bottom
          left: 0
          width: 100%
        .top
          top: 0
        .bottom
          bottom: 0
        .left
          left: 0
        .right
          right: 0
        .box
          width: 0
          height: 0
          background: transparent
          opacity: 1
          border: 2px solid #09c
          z-index: 4
          .canvas
            width: 100%
            height: 100%
            cursor: move
          .board
            position: absolute
            right: 0
            bottom: -52px
            width: 323px
            height: 50px
            background: #fff
            .tool
              display: flex
              justify-content: space-around
              align-items: center
              height: 100%
              .list
                display: inline-flex
                justify-content: center
                align-items: center
                /*width: 32px*/
                /*height: 32px*/
                padding: 5px
                cursor: pointer
                box-sizing: border-box
                border-radius: 5px
                &:hover, &.active
                  background: #ddd
                .iconfont
                  font-size: 20px
                  &.icon-arrow, &.icon-font
                    color: #09c
                  &.icon-close
                    color: red
                  &.icon-check
                    color: green
                .text
                  margin-left: 5px
                  font-size: 14px
            .setting
              display: flex
              align-items: center
              height: 100%
              margin-top: 1px
              padding: 3px
              background: #e4eff1
              border-radius: 3px
              box-sizing: border-box
              .left
                padding: 5px 8px 5px 0
                margin-right: 5px
                border-right: 1px solid #dadada
                .line
                  .list
                    display: inline-flex
                    justify-content: center
                    align-items: center
                    float: left
                    width: 28px
                    height: 28px
                    margin-left: 5px
                    border-radius: 3px
                    cursor: pointer
                    box-sizing: border-box
                    &.active
                      background: #c3d8dc
                      border: 1px solid #b0cbd0
                    &:hover
                      border: 1px solid #b0cbd0
                    .round
                      border-radius: 50%
                      background: #666
                      &.round1
                        width: 4px
                        height: 4px
                      &.round2
                        width: 8px
                        height: 8px
                      &.round3
                        width: 12px
                        height: 12px
              .color
                display: flex
                align-items: center
                .show
                  width: 30px
                  height: 30px
                  margin: 2px
                  background: #63a35c
                  border: 1px solid #000000
                  border-radius: 3px
                  box-sizing: border-box
                .color-list
                  display: flex
                  flex-wrap: wrap
                  width: 170px
                  .list
                    width: 15px
                    height: 15px
                    margin: 3px
                    background: #63a35c
                    box-sizing: border-box
                    border-radius: 3px
                    cursor: pointer
                    &:hover
                      border: 1px solid #000000
          .direction
            position: absolute
            width: 6px
            height: 6px
            background: #09c
            &.nw, &.n, &.ne
              top: -4px
            &.nw, &.sw, &.w
              left: -4px
            &.n, &.s
              left: 50%
              cursor: ns-resize
            &.ne, &.e, &.se
              right: -4px
            &.e, &.w
              top: 50%
              cursor: ew-resize
            &.se, &.s, &.sw
              bottom: -4px
            &.nw, &.se
              cursor: nw-resize
            &.ne, &.sw
              cursor: ne-resize
</style>

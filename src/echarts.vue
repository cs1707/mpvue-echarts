<template>
  <canvas
    v-if="canvasId"
    class="ec-canvas"
    :canvasId="canvasId"
    @touchstart="touchStart"
    @touchmove="touchMove"
    @touchend="touchEnd"
    @error="error">
  </canvas>
</template>

<script>
import WxCanvas from './wx-canvas';
import * as echarts from 'echarts';

let ctx;
let chart

export default {
  props: {
    canvasId: {
      type: String,
      default: 'ec-canvas'
    },

    ec: {
      type: Object
    }
  },

  data() {
    return {
      chart: null
    }
  },

  onReady () {
    if (!this.ec) {
      console.warn('组件需绑定 ec 变量，例：<ec-canvas id="mychart-dom-bar" '
        + 'canvas-id="mychart-bar" ec="{{ ec }}"></ec-canvas>');
      return;
    }

    if (!this.ec.lazyLoad) {
      setTimeout(() => {
        this.init();
      }, 50);
    }
  },

  methods: {
    init: function (callback) {
      const version = wx.version.version.split('.').map(n => parseInt(n, 10));
      const isValid = version[0] > 1 || (version[0] === 1 && version[1] > 9)
        || (version[0] === 1 && version[1] === 9 && version[2] >= 91);
      if (!isValid) {
        console.error('微信基础库版本过低，需大于等于 1.9.91。'
          + '参见：https://github.com/ecomfe/echarts-for-weixin'
          + '#%E5%BE%AE%E4%BF%A1%E7%89%88%E6%9C%AC%E8%A6%81%E6%B1%82');
        return;
      }

      ctx = wx.createCanvasContext(this.canvasId, this);

      const canvas = new WxCanvas(ctx, this.canvasId);

      echarts.setCanvasCreator(() => {
        return canvas;
      });

      var query = wx.createSelectorQuery()
      query.select('.ec-canvas').boundingClientRect(res => {
        if (typeof callback === 'function') {
          chart = callback(canvas, res.width, res.height);
        }
        else if (this.ec && this.ec.onInit) {
          chart = this.ec.onInit(canvas, res.width, res.height);
        }
      }).exec();
    },

    canvasToTempFilePath(opt) {
      if (!opt.canvasId) {
        opt.canvasId = this.canvasId;
      }

      ctx.draw(true, () => {
        wx.canvasToTempFilePath(opt, this);
      });
    },

    touchStart(e) {
      console.log('start')
      if (!this.ec.disableTouch && chart &&e.mp.touches.length > 0) {
        var touch =e.mp.touches[0];
        console.log(touch.x, touch.y)
        chart._zr.handler.dispatch('mousedown', {
          zrX: touch.x,
          zrY: touch.y
        });
        chart._zr.handler.dispatch('mousemove', {
          zrX: touch.x,
          zrY: touch.y
        });
      }
    },

    touchMove(e) {
      console.log('move')
      if (!this.ec.disableTouch && chart &&e.mp.touches.length > 0) {
        var touch =e.mp.touches[0];
        chart._zr.handler.dispatch('mousemove', {
          zrX: touch.x,
          zrY: touch.y
        });
      }
    },

    touchEnd(e) {
      setTimeout(() => {
        if (!this.ec.disableTouch && chart) {
          const touch = e.changedTouches ? e.changedTouches[0] : {};
          chart._zr.handler.dispatch('mouseup', {
            zrX: touch.x,
            zrY: touch.y
          });
          chart._zr.handler.dispatch('click', {
            zrX: touch.x,
            zrY: touch.y
          });
        }
      }, 50);
    }
  }
}
</script>

<style scoped>
.ec-canvas {
  width: 100%;
  height: 100%;
}
</style>

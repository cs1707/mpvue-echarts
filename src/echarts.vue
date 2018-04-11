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
import * as echarts from 'echarts';
import WxCanvas from './wx-canvas';

let chart;

export default {
  props: {
    ec: {
      type: Object,
      default() {
        return {}
      }
    },
    canvasId: {
      type: String,
      default() {
        return 'ec-canvas'
      }
    }
  },
  data () {
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
      setTimeout(() => this.init(), 50);
    }
  },
  methods: {
    init () {
      const version = wx.version.version.split('.').map(n => parseInt(n, 10));
      const isValid = version[0] > 1 || (version[0] === 1 && version[1] > 9)
        || (version[0] === 1 && version[1] === 9 && version[2] >= 91);
      if (!isValid) {
        console.error('微信基础库版本过低，需大于等于 1.9.91。'
          + '参见：https://github.com/ecomfe/echarts-for-weixin'
          + '#%E5%BE%AE%E4%BF%A1%E7%89%88%E6%9C%AC%E8%A6%81%E6%B1%82');
        return;
      }

      if (!this.ec.onInit) {
        console.warn('请传入 onInit 函数进行初始化');
        return;
      }

      const { canvasId } = this;
      const ctx = wx.createCanvasContext(canvasId);

      const canvas = new WxCanvas(ctx);

      echarts.setCanvasCreator(() => {
        return canvas;
      });

      var query = wx.createSelectorQuery();
      query.select('.ec-canvas').boundingClientRect(res => {
        if (!res) {
          setTimeout(() => this.init(), 50);
          return;
        }
        chart = this.ec.onInit(canvas, res.width, res.height);
      }).exec();
    },
    touchStart (e) {
      if (chart && e.mp.touches.length > 0) {
        var touch = e.mp.touches[0];
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
    touchMove (e) {
      if (chart && e.mp.touches.length > 0) {
        var touch = e.mp.touches[0];
        chart._zr.handler.dispatch('mousemove', {
          zrX: touch.x,
          zrY: touch.y
        });
      }
    },
    touchEnd (e) {
      setTimeout(() => {
        if (chart) {
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

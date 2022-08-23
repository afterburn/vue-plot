<template>
  <svg
    id="chart"
    :width="width"
    :height="height"
    :viewBox="`0 0 ${width} ${height}`"
    xmlns="http://www.w3.org/2000/svg"
  >
    <defs>
      <linearGradient :id="id" x1="0%" y1="0%" x2="0%" y2="100%">
        <stop offset="0%" :style="`stop-color: ${bgColor}; stop-opacity: 1`" />
        <stop
          offset="100%"
          :style="`stop-color: ${bgColor}; stop-opacity: 0`"
        />
      </linearGradient>
    </defs>
    <path
      :d="`${path} L ${width} ${height},L 0 ${height}Z`"
      :fill="background"
    />
    <path :d="path" :stroke="color" :stroke-width="strokeWidth" fill="none" />
    <g>
      <circle
        v-for="(p, i) in data"
        :key="i"
        :cx="p[0]"
        :cy="p[1]"
        :r="circleRadius"
        :fill="color"
      />
    </g>
  </svg>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  props: {
    sm: {
      type: Boolean,
      default: false,
    },

    color: {
      type: String,
      default: 'black',
    },

    bgColor: {
      type: String,
      default: 'black',
    },

    bgType: {
      type: String,
      default: 'gradient'
    },

    data: {
      type: Array,
      required: true,
    },

    smoothness: {
      type: Number,
      default: 0.2,
    },

    strokeWidth: {
      type: Number,
      default: 2,
    },

    circleRadius: {
      type: Number,
      default: 0,
    },
  },

  mounted() {
    this.resizeObserver = new ResizeObserver((entries) => {
      for (const entry of entries) {
        if (entry.contentRect) {
          const { width, height } = entry.contentRect;
          this.width = width;
          this.height = height;
        }
      }
    });

    this.observedElement = this.$el.parentElement;
    this.resizeObserver.observe(this.observedElement as Element);
  },

  beforeDestroy() {
    this.resizeObserver.unobserve(this.observedElement as Element);
  },

  data() {
    return {
      id: Math.random(),
      width: 0,
      height: 0,
      observedElement: undefined as unknown as HTMLElement | null,
      resizeObserver: undefined as unknown as ResizeObserver
    };
  },

  computed: {
    path(): string {
      const { data, width, height } = this;

      // Normalize data
      const maxX = Math.max(...data.map((p: any) => p[0]));
      const maxY = Math.max(...data.map((p: any) => p[1]));
      const ratioX = maxX / width;
      const ratioY = maxY / height;
      const normalizedData = data.map((p: any) => {
        return [this.normalize(p[0], ratioX), this.normalize(p[1], ratioY)];
      });

      // Create the path
      return normalizedData.reduce(
        (acc, point, i, a) =>
          i === 0
            ? `M ${point[0]},${point[1]}`
            : `${acc} ${this.curve(point, i, a)}`,
        ""
      );
    },

    background(): string {
      const { bgType, bgColor, id } = this

      switch(bgType) {
        case 'gradient': return `url(#${id})`
        case 'solid': return bgColor
        case 'transparent': return 'transparent'
        default: return ''
      }
    }
  },

  methods: {
    normalize(v: number, ratio: number) {
      return v / ratio;
    },

    curve(point: number[], i: number, a: number[][]) {
      const cp1 = this.cp(a[i - 1], a[i - 2], point);
      const cp2 = this.cp(point, a[i - 1], a[i + 1], true);
      return `C ${cp1[0]},${cp1[1]} ${cp2[0]},${cp2[1]} ${point[0]},${point[1]}`;
    },

    line(pointA: number[], pointB: number[]) {
      const lengthX = pointB[0] - pointA[0];
      const lengthY = pointB[1] - pointA[1];
      return {
        length: Math.sqrt(Math.pow(lengthX, 2) + Math.pow(lengthY, 2)),
        angle: Math.atan2(lengthY, lengthX),
      };
    },

    cp(current: number[], previous: number[], next: number[], reverse?: boolean) {
      const p = previous || current;
      const n = next || current;
      const o = this.line(p, n);
      const angle = o.angle + (reverse ? Math.PI : 0);
      const length = o.length * this.smoothness;
      const x = current[0] + Math.cos(angle) * length;
      const y = current[1] + Math.sin(angle) * length;
      return [x, y];
    },
  },
})
</script>
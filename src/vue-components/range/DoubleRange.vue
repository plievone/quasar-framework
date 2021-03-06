<template>
  <div
    class="quasar-range non-selectable"
    :class="{disabled: disable}"
    @mousedown.prevent="__setActive"
    @touchstart.prevent="__setActive"
    @touchend.prevent="__end"
    @touchmove.prevent="__update"
  >
    <div ref="handle" class="quasar-range-handle-container">
      <div class="quasar-range-track"></div>
      <div
        v-if="markers"
        class="quasar-range-mark"
        v-for="n in ((max - min) / step + 1)"
        :style="{left: (n - 1) * 100 * step / (max - min) + '%'}"
      ></div>
      <div
        class="quasar-range-track active-track no-transition"
        :style="{left: percentageMin * 100 + '%', width: activeTrackWidth}"
      ></div>
      <div
        class="quasar-range-handle no-transition"
        :style="{left: percentageMin * 100 + '%'}"
        :class="{dragging: dragging, 'handle-at-minimum': value.min === min}"
      >
        <div
          class="quasar-range-label"
          v-if="label"
        >{{ value.min }}</div>
      </div>
      <div
        class="quasar-range-handle no-transition"
        :style="{left: percentageMax * 100 + '%'}"
        :class="{dragging: dragging}"
      >
        <div
          class="quasar-range-label"
          v-if="label"
        >{{ value.max }}</div>
      </div>
    </div>
  </div>
</template>

<script>
import Utils from '../../utils'
import Platform from '../../platform'

export default {
  props: {
    value: {
      type: Object,
      required: true,
      validator (value) {
        return typeof value.min !== 'undefined' && typeof value.max !== 'undefined'
      }
    },
    min: {
      type: Number,
      default: 0
    },
    max: {
      type: Number,
      default: 100
    },
    step: {
      type: Number,
      default: 1
    },
    snap: Boolean,
    markers: Boolean,
    label: Boolean,
    disable: Boolean
  },
  data () {
    return {
      dragging: false,
      currentMinPercentage: (this.value.min - this.min) / (this.max - this.min),
      currentMaxPercentage: (this.value.max - this.min) / (this.max - this.min)
    }
  },
  computed: {
    percentageMin () {
      if (this.snap) {
        return (this.value.min - this.min) / (this.max - this.min)
      }
      return this.currentMinPercentage
    },
    percentageMax () {
      if (this.snap) {
        return (this.value.max - this.min) / (this.max - this.min)
      }
      return this.currentMaxPercentage
    },
    activeTrackWidth () {
      return 100 * (this.percentageMax - this.percentageMin) + '%'
    }
  },
  watch: {
    'value.min' (value) {
      if (this.dragging) {
        return
      }
      if (value > this.value.max) {
        value = this.value.max
      }
      this.currentMinPercentage = (value - this.min) / (this.max - this.min)
    },
    'value.max' (value) {
      if (this.dragging) {
        return
      }
      if (value < this.value.min) {
        value = this.value.min
      }
      this.currentMaxPercentage = (value - this.min) / (this.max - this.min)
    },
    min (value) {
      if (this.value.min < value) {
        this.____update({min: value})
      }
      if (this.value.max < value) {
        this.____update({max: value})
      }
      this.$nextTick(this.__validateProps)
    },
    max (value) {
      if (this.value.min > value) {
        this.____update({min: value})
      }
      if (this.value.max > value) {
        this.____update({max: value})
      }
      this.$nextTick(this.__validateProps)
    },
    step () {
      this.$nextTick(this.__validateProps)
    }
  },
  methods: {
    __setActive (event) {
      if (this.disable) {
        return
      }

      let container = this.$refs.handle

      this.dragging = {
        left: container.getBoundingClientRect().left,
        width: container.offsetWidth,
        valueMin: this.value.min,
        percentageMin: this.currentMinPercentage,
        valueMax: this.value.max,
        percentageMax: this.currentMaxPercentage
      }

      let
        offset = Utils.event.position(event).left - this.dragging.left,
        percentage = Math.min(1, Math.max(0, offset / this.dragging.width))

      this.dragging.onLeft = Math.abs(percentage - this.currentMinPercentage) <= Math.abs(percentage - this.currentMaxPercentage)
      this.__update(event)
    },
    __update (event) {
      if (!this.dragging) {
        return
      }

      let
        percentage = Math.min(1, Math.max(0, (Utils.event.position(event).left - this.dragging.left) / this.dragging.width)),
        model = this.min + percentage * (this.max - this.min),
        modulo = (model - this.min) % this.step

      model = Math.min(this.max, Math.max(this.min, model - modulo + (Math.abs(modulo) >= this.step / 2 ? (modulo < 0 ? -1 : 1) * this.step : 0)))

      if (this.dragging.onLeft) {
        if (percentage <= this.dragging.percentageMax) {
          this.currentMinPercentage = percentage
          this.currentMaxPercentage = this.dragging.percentageMax
          this.__updateInput({
            min: model,
            max: this.dragging.valueMax
          })
        }
        else {
          this.currentMinPercentage = this.dragging.percentageMax
          this.currentMaxPercentage = percentage
          this.__updateInput({
            min: this.dragging.valueMax,
            max: model
          })
        }
      }
      else {
        if (percentage >= this.dragging.percentageMin) {
          this.currentMaxPercentage = percentage
          this.currentMinPercentage = this.dragging.percentageMin
          this.__updateInput({
            min: this.dragging.valueMin,
            max: model
          })
        }
        else {
          this.currentMaxPercentage = this.dragging.percentageMin
          this.currentMinPercentage = percentage
          this.__updateInput({
            min: model,
            max: this.dragging.valueMin
          })
        }
      }
    },
    __updateInput ({min = this.value.min, max = this.value.max}) {
      this.$emit('input', {min, max})
    },
    __end () {
      this.dragging = false
      this.currentMinPercentage = (this.value.min - this.min) / (this.max - this.min)
      this.currentMaxPercentage = (this.value.max - this.min) / (this.max - this.min)
    },
    __validateProps () {
      if (this.min >= this.max) {
        console.error('Range error: min >= max', this.$el, this.min, this.max)
      }
      else if ((this.max - this.min) % this.step !== 0) {
        console.error('Range error: step must be a divisor of max - min', this.$el, this.min, this.max, this.step)
      }
      else if ((this.value.min - this.min) % this.step !== 0) {
        console.error('Range error: step must be a divisor of initial value.min - min', this.$el, this.value.min, this.min, this.step)
      }
      else if ((this.value.max - this.min) % this.step !== 0) {
        console.error('Range error: step must be a divisor of initial value.max - min', this.$el, this.value.max, this.max, this.step)
      }
    }
  },
  created () {
    this.__validateProps()
    if (Platform.is.desktop) {
      document.body.addEventListener('mousemove', this.__update)
      document.body.addEventListener('mouseup', this.__end)
    }
  },
  beforeDestroy () {
    if (Platform.is.dekstop) {
      document.body.removeEventListener('mousemove', this.__update)
      document.body.removeEventListener('mouseup', this.__end)
    }
  }
}
</script>

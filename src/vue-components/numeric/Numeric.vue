<template>
  <div class="quasar-numeric textfield row inline items-center" :class="{disabled: disable}">
    <i @click="__setByOffset(-1)">remove</i>
    <input
      class="no-style auto quasar-input-field"
      type="number"
      v-model.number="model"
      @blur="__updateValue()"
      @keydown.enter="__updateValue()"
      @keydown.up="__setByOffset(1)"
      @keydown.down="__setByOffset(-1)"
      @keydown.esc="model = value"
      :disabled="disable"
      :style="{width: (''+model).length * .7 + 'em'}"
    >
    <i v-show="value !== model && model !== ''">check</i>
    <i @click="__setByOffset(1)">add</i>
  </div>
</template>

<script>
export default {
  props: {
    value: {
      type: Number,
      default: 0
    },
    step: {
      type: Number,
      default: 1
    },
    min: Number,
    max: Number,
    disable: Boolean
  },
  watch: {
    value () {
      this.model = this.value
    }
  },
  data () {
    return {
      model: this.value
    }
  },
  methods: {
    __normalize (value) {
      if (typeof this.min === 'number' && value < this.min) {
        return this.min
      }
      else if (typeof this.max === 'number' && value > this.max) {
        return this.max
      }
      return value
    },
    __updateValue () {
      this.model = this.__normalize(this.model)
      if (!this.disable && this.value !== this.model) {
        this.$emit('input', this.model)
      }
    },
    __setByOffset (direction) {
      if (!this.disable) {
        let newValue = this.model + direction * this.step
        if (typeof this.min === 'number' && newValue < this.min && this.model === this.min) {
          return
        }
        if (typeof this.max === 'number' && newValue > this.max && this.model === this.max) {
          return
        }
        this.model = newValue
        this.__updateValue()
      }
    }
  },
  created () {
    this.__updateValue()
  }
}
</script>

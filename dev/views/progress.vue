<template>
  <h1>Determinate</h1>
  <p>
    <quasar-progress :model="progress" @click="randomize()"></quasar-progress>
    <quasar-progress :model="progress" class="stripe" @click="randomize()"></quasar-progress>
    <quasar-progress :model="progress" class="stripe positive" @click="randomize()"></quasar-progress>
    <quasar-progress :model="progress" class="stripe info" @click="randomize()"></quasar-progress>
    <quasar-progress :model="progress" class="stripe warning" @click="randomize()"></quasar-progress>
    <quasar-progress :model="progress" class="stripe secondary" @click="randomize()"></quasar-progress>
    <quasar-progress :model="progress" class="stripe tertiary" @click="randomize()"></quasar-progress>
    <quasar-progress :model="progress" class="stripe dark" @click="randomize()"></quasar-progress>
    <quasar-progress :model="progress" class="stripe negative" @click="randomize()"></quasar-progress>
    <quasar-progress :model="progress" class="stripe animate" @click="randomize()"></quasar-progress>
  </p>

  <h1>Indeterminate</h1>
  <p>
    <quasar-progress class="indeterminate"></quasar-progress>
    <quasar-progress class="indeterminate positive"></quasar-progress>
    <quasar-progress class="indeterminate warning"></quasar-progress>
    <quasar-progress class="indeterminate negative"></quasar-progress>
    <quasar-progress class="indeterminate dark"></quasar-progress>
  </p>

  <h1>Specific Height</h1>
  <p>
    <quasar-progress class="indeterminate stripe" style="height: 25px"></quasar-progress>
  </p>
</template>

<script>
export default {
  data () {
    return {
      progress: 81
    }
  },
  methods: {
    randomize () {
      this.progress = Math.round(Math.random() * 100)
    }
  }
}
</script>

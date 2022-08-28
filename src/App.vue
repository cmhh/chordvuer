<script setup>
  import GuitarChordTransitions from './components/GuitarChordTransitions.vue'
  import { chords } from './chords.js'
</script>

<script>
export default {
  data () {
    return {
      selected: null,
      selections: [],
      speed: 1
    }
  },
  watch: {
    selected() {
      this.selections.push(chords[this.selected])
    }
  },
  computed: {
    selectionStr() {
      if (this.selections.length === 0) return ""
      return this.selections.map((x) => x.name).join(", ")
    }
  },
  methods: {
    reset() {
      this.selections = []
    }
  }
}
</script>

<template>
<div>
  <label for="chord-name">Chord: </label>
  <select v-model="selected" >
    <option v-for="(v, k) in chords" :value="k" :key="k" >
      {{ v.name }}
    </option>
  </select>
  <br>
  <label for="dur">Transition speed (seconds): </label>
  <input type="number" min="0" max="5" v-model="speed">
  <br>
  Selections: {{ selectionStr }}
  <br>
  <button @click="reset()" >reset</button>
  <hr>
  <GuitarChordTransitions 
    id="chord3" 
    :duration="this.speed * 1000" 
    :chords="selections"
  ></GuitarChordTransitions>
</div> 
</template>

<style scoped>
</style>
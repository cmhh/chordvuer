<script setup>
  import Chord from './components/Chord.vue'
  import { chords } from './chords.js'
</script>

<script>
export default {
  data() {
    return {
      selected: "A"
    }
  },
  ready() {

  }
}
</script>


<template>
<div>
<label for="chord-name">Choose a chord: </label>
<select v-model="selected">
  <option v-for="(v, k) in chords" :value="k" :key="k" :selected="selected" >
   {{ v.name }}
  </option>
</select>
<hr>
<Chord id="chord1" v-bind:chord="chords[this.selected]" />
</div> 

</template>

<style scoped>
</style>

<script setup>
  import Chord from './components/Chord.vue'
  import { chords } from './chords.js'
</script>

<script>
export default {
  data() {
    return {chordname: null}
  },
  computed: {
    chordInfo() {
      return chords[this.chordname]
    }
  }
}
</script>


<template>
<Chord id="chord1" :chord="chords.Em" />
<Chord id="chord2" :chord="chords.Gmaj" />


<div>
<div>Selected: {{ chordname }}</div>
<label for="chord-name">Choose a chord:</label>
<select v-model="chordname">
  <option value="Em" selected>E minor</option>
  <option value="Gmaj">G major</option>
</select>
<hr>
<Chord id="chord3" v-bind:chord="chordInfo" />
</div>

</template>

<style scoped>
</style>

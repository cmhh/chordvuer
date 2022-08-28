<script>
  import GuitarChord from './GuitarChord.vue'

  export default {
    name: "GuitarChordTransitions",
    components: {
      GuitarChord
    },
    data () {
      return {
        chord: null
      }
    },
    props: {
      id: {
        type: String,
        required: true
      },
      duration: {
        type: Number, 
        default: 1000
      },
      fps: {
        type: Number,
        default: 10
      },
      chords: {
        type: Array,
        default: []
      }
    },
    mounted() {
      this.loop()
    },
    methods: {
      async loop() {
        var i = 1
        while (true) {
          await(this.sleep(this.duration))
          this.chord = this.chords[i]
          i += 1
          if (i >= this.chords.length) i = 0
        }
      },
      sleep(ms) {
        return new Promise(resolve => setTimeout(resolve, ms))
      }
    }
  }
</script>

<template>
  <GuitarChord :id="`${id}-chord`" :duration="duration / 2" :fps="fps" :chord="chord" />
</template>
<script>
  export default {
    name: "Chord",
    data() {
      return {current: null}
    },
    props: {
      id: {
        type: String,
        required: true
      },
      transitionSpeed: {
        type: Number,
        default: 1000
      },
      duration: {
        type: Number, 
        default: 1000
      },
      fps: {
        type: Number,
        default: 10
      },
      chord: {
        type: Object,
        required: true
      }
    },
    computed: {
      frames() {
        const s = this.duration / 1000
        return Math.ceil(this.fps * s)
      },
      frameLength() {
        return 1000 / this.fps
      }
    },
    watch: {
      chord(newVal, oldVal) {
        this.applyAll(newVal, oldVal)
      }
    },
    mounted() {
      this.applyAll(this.chord, null)
    },
    methods: {
      async applyAll(newVal, oldVal) {
        if (!newVal) return
        this.applyTitle(newVal, oldVal)
        this.applyFingers(newVal)
        this.applyStrike()
        this.applyNotes()      
        this.current = this.chord
      },
      async applyTitle(newVal, oldVal) {
        const svg = document.getElementById(this.id)
        var name = svg.getElementsByClassName("chord-name")[0]
        if (oldVal) this.swaptxt(name, newVal.name)
        else {
          name.setAttributeNS(null, "opacity", 0)
          name.textContent = newVal.name
          this.fadein(name)
        }
      },
      async applyFingers(newVal) {
        [1,2,3,4].forEach((x) => this.applyFinger(x, newVal))
      },
      async applyFinger(fingerno, newVal) {
        const svg = document.getElementById(this.id)

        const prevshp = svg.querySelectorAll(`circle.finger-${fingerno}`)
        const prevtxt = svg.querySelectorAll(`text.finger-${fingerno}`)

        const thisfinger = newVal.fingers.filter((x) => x.finger == fingerno)[0]

        if (prevshp.length > 0 && !thisfinger) {           // fade out
          if (fingerno == 2) console.log("fading out, man.")
          prevshp.forEach((x) => this.fadeout(x))
          prevtxt.forEach((x) => this.fadeout(x))
        } else if (prevshp.length == 0 && thisfinger) {    // fade in
          thisfinger.strings.map((x) => {
            const [c, l] = this.makeFinger(fingerno, x, thisfinger.fret)
            svg.appendChild(c)
            svg.appendChild(l)
            this.fadein(c)
            this.fadein(l)
          })
        } else if (prevshp.length > 0 && thisfinger) {     // move
          // fade out strings, ex. 1st listed
          if (prevshp.length > 1) {
            for (var i=1; i < prev.length; i++) {
              this.fadeout(prevshp[i])
              this.fadeout(prevtxt[i])
            }
          }

          // fade in strings, ex. 1st listed
          if (thisfinger.strings > 1) {
            thisfinger.strings.slice(1).map((x) => {
              const [c, l] = this.makeFinger(fingerno, x, thisfinger.fret)
              svg.appendChild(c)
              svg.appendChild(l)
              this.fadein(c)
              this.fadein(l)
            })
          }

          // move first listed
          this.move(prevshp[0], thisfinger.strings[0], thisfinger.fret)
          this.move(prevtxt[0], thisfinger.strings[0], thisfinger.fret)
        } 
      },
      makeFinger(fingerno, string, fret) {
        const ns = "http://www.w3.org/2000/svg"
        const xy = this.XY(string, fret)

        const circ = document.createElementNS(ns, "circle")
        circ.setAttributeNS(null, "class", `finger finger-${fingerno}`)
        circ.setAttributeNS(null, "cx", xy[0])
        circ.setAttributeNS(null, "cy", xy[1])
        circ.setAttributeNS(null, "r", "0.3")
        circ.setAttributeNS(null, "opacity", 0)

        const lbl = document.createElementNS(ns, "text")
        lbl.setAttributeNS(null, "class", `finger finger-${fingerno}`)
        lbl.setAttributeNS(null, "x", xy[0])
        lbl.setAttributeNS(null, "y", xy[1])
        lbl.setAttributeNS(null, "text-anchor", "middle")
        lbl.setAttributeNS(null, "alignment-baseline", "middle")
        lbl.setAttributeNS(null, "opacity", 0)
        lbl.textContent = fingerno

        return [circ, lbl]
      },
      async applyStrike() {
        if (!this.chord) return null
        const svg = document.getElementById(this.id)
        const els = Array.prototype.slice.call(svg.getElementsByClassName("strike"))

        const newlbl = [1,2,3,4,5,6].map((x) => {
          if (this.chord.open.includes(x)) return "O"
          if (this.chord.closed.includes(x)) return "X"
          return ""
        })

        els.forEach((el, i) => {
          if (el.textContent != newlbl[i]) this.swaptxt(el, newlbl[i])
        })
      },
      async applyNotes() {
        const svg = document.getElementById(this.id)
        const els = Array.prototype.slice.call(svg.getElementsByClassName("note"))

        const newlbl = [1,2,3,4,5,6].map((x) => {
          const z = this.chord.fingers.filter((y) => y.strings.includes(x)).map((y) => y.fret)
          if (z.length == 0) return this.note(x, 0)
          else return this.note(x, Math.max(...z))
        })

        els.forEach((el, i) => {
          if (el.textContent != newlbl[i]) this.swaptxt(el, newlbl[i])
        })
      },
      sleep(ms) {
        return new Promise(resolve => setTimeout(resolve, ms))
      },
      XY(string, fret) {
        const X = 6 - string + 1
        const Y = 1 + (1.5 / 2) + (fret - 1) * (1.5)
        return [X, Y]
      },
      note(string, fret) {
        function next(note) {
          if (note == 'A') return 'A#'
          else if (note == 'A#') return 'B'
          else if (note == 'B')  return 'C'
          else if (note == 'C')  return 'C#'
          else if (note == 'C#') return 'D'
          else if (note == 'D')  return 'D#'
          else if (note == 'D#') return 'E'
          else if (note == 'E')  return 'F'
          else if (note == 'F')  return 'F#'
          else if (note == 'F#') return 'G'
          else if (note == 'G')  return 'G#'
          else if (note == 'G#') return 'A'
        }

        function start(string) {
          if (string == 6) return "E"
          else if (string == 5) return "A"
          else if (string == 4) return "D"
          else if (string == 3) return "G"
          else if (string == 2) return "B"
          else if (string == 1) return "E"
        }

        var res = start(string)
        for (var i = 0; i < fret; i++) res = next(res)
        return res
      },
      seq(from, to, steps) {
        const dd = (to - from) / steps
        const res = Array(steps - 1).fill().map((e, i) => from + dd * (i + 1)) 
        res.push(to)
        return res
      },
      path(pos1, pos2, steps) {
        const xs = this.seq(pos1[0], pos2[0], steps)
        const ys = this.seq(pos1[1], pos2[1], steps)
        return xs.map((e, i) => [e, ys[i]])
      },
      async fadeout(el) {
        for (var i = 0; i < this.frames; i++) {
          await this.sleep(this.frameLength)
          el.setAttributeNS(null, "opacity", 1 - (i + 1) * (1 / this.frames))
        }
        document.getElementById(this.id).removeChild(el)
      },
      async fadein(el) {
        for (var i = 0; i < this.frames; i++) {
          await this.sleep(this.frameLength)
          el.setAttributeNS(null, "opacity", (i + 1) * (1 / this.frames))
        }
      },
      async swaptxt(el, str) {
        const n1 = Math.floor(this.frames / 2)
        const n2 = this.frames - n1
        for (var i = 0; i < n1; i++) {
          await this.sleep(this.frameLength)
          el.setAttributeNS(null, "opacity", 1 - (i + 1) * (1 / n1))
        }

        el.textContent = str

        for (var i = 0; i < n2; i++) {
          await this.sleep(this.frameLength)
          el.setAttributeNS(null, "opacity", (i + 1) * (1 / n2))
        }
      },
      async move(el, string, fret) {
        const x = el.tagName == "circle" ? "cx" : "x"
        const y = el.tagName == "circle" ? "cy" : "y"

        const p0 = [
          parseFloat(el.getAttributeNS(null, x)), 
          parseFloat(el.getAttributeNS(null, y))
        ]

        const p1 = this.XY(string, fret)
        const path = this.path(p0, p1, this.frames)

        for (var i=0; i < path.length; i++) {
          await this.sleep(this.frameLength)
          el.setAttributeNS(null, x, path[i][0])
          el.setAttributeNS(null, y, path[i][1])
        }
      }
    }
  }
</script>

<template>
  <svg :id="id" class="chord" viewBox="0 0 7 9.5" > 
    <!-- static objects -->
    <rect class="fretboard" x="1" y="1" width="5" height="7.5" />

    <line class="nut"  x1="1" y1="1"  x2="6" y2="1" />

    <line class="fret" x1="1" y1="2.5" x2="6" y2="2.5" />
    <line class="fret" x1="1" y1="4"   x2="6" y2="4" />
    <line class="fret" x1="1" y1="5.5" x2="6" y2="5.5" />
    <line class="fret" x1="1" y1="7"   x2="6" y2="7" />
    <line class="fret" x1="1" y1="8.5" x2="6" y2="8.5" />

    <line class="string string-high-e" x1="6" y1="1" x2="6" y2="8.5" />
    <line class="string string-b"      x1="5" y1="1" x2="5" y2="8.5" />
    <line class="string string-g"      x1="4" y1="1" x2="4" y2="8.5" />
    <line class="string string-d"      x1="3" y1="1" x2="3" y2="8.5" />
    <line class="string string-a"      x1="2" y1="1" x2="2" y2="8.5" />
    <line class="string string-low-e"  x1="1" y1="1" x2="1" y2="8.5" />

    <!-- dynamic objects -->
    <text class="chord-name" x="3.5" y="0" text-anchor="middle" alignment-baseline="hanging"></text>
    <text class="strike strike-high-e" x="6" y="1" dy="-0.2" text-anchor="middle" alignment-baseline="top"></text>
    <text class="strike strike-b"      x="5" y="1" dy="-0.2" text-anchor="middle" alignment-baseline="top"></text>
    <text class="strike strike-g"      x="4" y="1" dy="-0.2" text-anchor="middle" alignment-baseline="top"></text>
    <text class="strike strike-d"      x="3" y="1" dy="-0.2" text-anchor="middle" alignment-baseline="top"></text>
    <text class="strike strike-a"      x="2" y="1" dy="-0.2" text-anchor="middle" alignment-baseline="top"></text>
    <text class="strike strike-low-e"  x="1" y="1" dy="-0.2" text-anchor="middle" alignment-baseline="top"></text>

    <text class="note note-high-e" x="6" y="8.5" dy="0.2" text-anchor="middle" alignment-baseline="hanging">E</text>
    <text class="note note-b"      x="5" y="8.5" dy="0.2" text-anchor="middle" alignment-baseline="hanging">B</text>
    <text class="note note-g"      x="4" y="8.5" dy="0.2" text-anchor="middle" alignment-baseline="hanging">G</text>
    <text class="note note-d"      x="3" y="8.5" dy="0.2" text-anchor="middle" alignment-baseline="hanging">D</text>
    <text class="note note-a"      x="2" y="8.5" dy="0.2" text-anchor="middle" alignment-baseline="hanging">A</text>
    <text class="note note-low-e"  x="1" y="8.5" dy="0.2" text-anchor="middle" alignment-baseline="hanging">E</text>
  </svg>
</template>

<style scoped>
  @import url('https://fonts.googleapis.com/css?family=Source+Sans+Pro');

  .chord * {
    font-family: "Source Sans Pro", Helvetica, sans-serif;
  }

  .chord-name {
    font-size: 3%;
  }

  .fretboard {
    fill: #FFFFCC;
  }

  line {
    stroke-width: 0.1;
    stroke: black;
    stroke-linecap: round;
  }

  .nut {
    stroke-width: 0.2
  }

  .fret {
    stroke: grey;
  }

  :deep(.finger-1) {
    fill: #FFB000;
  }

  :deep(.finger-2) {
    fill: #FE6100;
  }

  :deep(.finger-3) {
    fill: #DC267F;
  }

  :deep(.finger-4) {
    fill: #785EF0;
  }

  :deep(.finger-5) {
    fill: #648FFF;
  }

  :deep(text.finger) {
    font-size: 2%;
    font-weight: bold;
    fill: #FFFFFF;
  }

  text.strike, text.note {
    font-size: 2%;
  }
</style>

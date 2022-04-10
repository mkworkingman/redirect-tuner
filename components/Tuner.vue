<template>
  <div>
    <h2>Tuner:</h2>
    <h3>Frequency: {{this.frequency}} <span v-show="frequency">Hz</span></h3>
    <p>{{this.beforeActiveNote}} {{this.activeNote}} {{this.afterActiveNote}}</p>
  </div>
</template>

<script>
export default {
  props: {
    fileName: String
  },
  data() {
    return {
      audioUrl: null,
      audioSource: null,
      audioContext: null,
      analyser: null,
      mic: null,
      source: null,
      fftSize: 2048,
      dataArray: null,

      listening: true,
      keysSharp: ['C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#', 'A', 'A#', 'B'],
      keysFlat: ['C', 'Db', 'D', 'Eb', 'E', 'F', 'Gb', 'G', 'Ab', 'A', 'Bb', 'B'],

      noteNamesSharp: ['C0', 'C#0', 'D0', 'D#0', 'E0', 'F0', 'F#0', 'G0', 'G#0', 'A0', 'A#0', 'B0', 'C1', 'C#1', 'D1', 'D#1', 'E1', 'F1', 'F#1', 'G1', 'G#1', 'A1', 'A#1', 'B1', 'C2', 'C#2', 'D2', 'D#2', 'E2', 'F2', 'F#2', 'G2', 'G#2', 'A2', 'A#2', 'B2', 'C3', 'C#3', 'D3', 'D#3', 'E3', 'F3', 'F#3', 'G3', 'G#3', 'A3', 'A#3', 'B3', 'C4', 'C#4', 'D4', 'D#4', 'E4', 'F4', 'F#4', 'G4', 'G#4', 'A4', 'A#4', 'B4', 'C5', 'C#5', 'D5', 'D#5', 'E5', 'F5', 'F#5', 'G5', 'G#5', 'A5', 'A#5', 'B5', 'C6', 'C#6', 'D6', 'D#6', 'E6', 'F6', 'F#6', 'G6', 'G#6', 'A6', 'A#6', 'B6', 'C7', 'C#7', 'D7', 'D#7', 'E7', 'F7', 'F#7', 'G7', 'G#7', 'A7', 'A#7', 'B7', 'C8', 'C#8', 'D8', 'D#8'],
      noteNamesFlat: ['C0', 'Db0', 'D0', 'Eb0', 'E0', 'F0', 'Gb0', 'G0', 'Ab0', 'A0', 'Bb0', 'B0', 'C1', 'Db1', 'D1', 'Eb1', 'E1', 'F1', 'Gb1', 'G1', 'Ab1', 'A1', 'Bb1', 'B1', 'C2', 'Db2', 'D2', 'Eb2', 'E2', 'F2', 'Gb2', 'G2', 'Ab2', 'A2', 'Bb2', 'B2', 'C3', 'Db3', 'D3', 'Eb3', 'E3', 'F3', 'Gb3', 'G3', 'Ab3', 'A3', 'Bb3', 'B3', 'C4', 'Db4', 'D4', 'Eb4', 'E4', 'F4', 'Gb4', 'G4', 'Ab4', 'A4', 'Bb4', 'B4', 'C5', 'Db5', 'D5', 'Eb5', 'E5', 'F5', 'Gb5', 'G5', 'Ab5', 'A5', 'Bb5', 'B5', 'C6', 'Db6', 'D6', 'Eb6', 'E6', 'F6', 'Gb6', 'G6', 'Ab6', 'A6', 'Bb6', 'B6', 'C7', 'Db7', 'D7', 'Eb7', 'E7', 'F7', 'Gb7', 'G7', 'Ab7', 'A7', 'Bb7', 'B7', 'C8', 'Db8', 'D8', 'Eb8'],
      noteFrequencies: [16.35, 17.32, 18.35, 19.45, 20.6, 21.83, 23.12, 24.5, 25.96, 27.5, 29.14, 30.87, 32.7, 34.65, 36.71, 38.89, 41.2, 43.65, 46.25, 49, 51.91, 55, 58.27, 61.74, 65.41, 69.3, 73.42, 77.78, 82.41, 87.31, 92.5, 98, 103.83, 110, 116.54, 123.47, 130.81, 138.59, 146.83, 155.56, 164.81, 174.61, 185, 196, 207.65, 220, 233.08, 246.94, 261.63, 277.18, 293.66, 311.13, 329.63, 349.23, 369.99, 392, 415.3, 440, 466.16, 493.88, 523.25, 554.37, 587.33, 622.25, 659.26, 698.46, 739.99, 783.99, 830.61, 880, 932.33, 987.77, 1046.5, 1108.73, 1174.66, 1244.51, 1318.51, 1396.91, 1479.98, 1567.98, 1661.22, 1760, 1864.66, 1975.53, 2093, 2217.46, 2349.32, 2489.02, 2637.02, 2793.83, 2959.96, 3135.96, 3322.44, 3520, 3729.31, 3951.07, 4186.01, 4434.92, 4698.64, 4978.03],

      frequency: null,
      frequenciesHistory: [],
      beforeActiveNote: null,
      activeNote: null,
      afterActiveNote: null
    }
  },
  methods: {
    async setup() {
      this.audioContext = new AudioContext()
      this.analyser = new AnalyserNode(this.audioContext, { fftSize: this.fftSize })
      this.mic = await navigator.mediaDevices.getUserMedia({
        audio: true
      })
      if (this.audioContext.state === 'suspended') {
        await this.audioContext.resume()
      }

      this.dataArray = new Float32Array(this.fftSize)

      this.source = this.audioContext.createMediaStreamSource(this.mic)
      this.source.connect(this.analyser)
    },
    test() {
      requestAnimationFrame(this.test)
      this.analyser.getFloatTimeDomainData(this.dataArray)

      const currentFrequency = this.detectFrequency()

      if (
        currentFrequency &&
        currentFrequency < 2000 &&
        Math.round(Math.abs(currentFrequency - this.frequency)) >= 5
      ) {
        this.frequency = currentFrequency

        for (const index in this.noteFrequencies) {
          if (currentFrequency < this.noteFrequencies[index]) {
            const prevNote = this.noteNamesSharp[+index - 1]
            const nextNote = this.noteNamesSharp[index]
            this.activeNote = currentFrequency < ((this.noteFrequencies[index] + this.noteFrequencies[+index - 1]) / 2)
              ? prevNote : nextNote

            this.beforeActiveNote = this.activeNote === nextNote ? prevNote : this.noteNamesSharp[+index - 2]
            this.afterActiveNote = this.activeNote === nextNote ? this.noteNamesSharp[+index + 1] : nextNote
            break
          }
        }
      }
    },
    detectFrequency() {
      let buffer = this.dataArray
      let size = buffer.length
      let sumOfSquares = 0
      for (let i = 0; i < size; i++) {
        let val = buffer[i]
        sumOfSquares += val * val
      }
      const rootMeanSquare = Math.sqrt(sumOfSquares / size)
      if (rootMeanSquare < 0.01) {
        return 0
      }

      let r1 = 0
      let r2 = size - 1
      const threshold = 0.2

      for (let i = 0; i < size / 2; i++) {
        if (Math.abs(buffer[i]) < threshold) {
          r1 = i
          break
        }
      }

      for (let i = 1; i < size / 2; i++) {
        if (Math.abs(buffer[size - i]) < threshold) {
          r2 = size - i
          break
        }
      }

      buffer = buffer.slice(r1, r2)
      size = buffer.length

      const c = new Array(size).fill(0)
      for (let i = 0; i < size; i++) {
        for (let j = 0; j < size - i; j++) {
          c[i] = c[i] + buffer[j] * buffer[j+i]
        }
      }

      let d = 0
      while (c[d] > c[d+1]) {
        d++
      }

      let maxValue = -1
      let maxIndex = -1
      for (let i = d; i < size; i++) {
        if (c[i] > maxValue) {
          maxValue = c[i]
          maxIndex = i
        }
      }

      let T0 = maxIndex
      const x1 = c[T0 - 1]
      const x2 = c[T0]
      const x3 = c[T0 + 1]

      const a = (x1 + x3 - 2 * x2) / 2
      const b = (x3 - x1) / 2
      if (a) {
        T0 = T0 - b / (2 * a)
      }

      return +(this.audioContext.sampleRate/T0).toFixed(1)
    }
  },
  async mounted() {
    await this.setup()
    this.test()
  },
  computed: {
    getNote() {

    }
  }
}
</script>

<style lang="postcss" scoped>

</style>
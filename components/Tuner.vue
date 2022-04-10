<template>
  <div>
    <h2>Tuner:</h2>
    <h3>Frequency: {{this.frequency}} <span v-show="frequency">Hz</span></h3>
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
      lastFrequencies: [],

      listening: true,
      keysSharp: ['C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#', 'A', 'A#', 'B'],
      keysFlat: ['C', 'Db', 'D', 'Eb', 'E', 'F', 'Gb', 'G', 'Ab', 'A', 'Bb', 'B'],
      frequency: null
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
        Math.round(Math.abs(currentFrequency - this.frequency)) >= 5
      ) {
        this.frequency = currentFrequency
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
}
</script>

<style lang="postcss" scoped>

</style>
<template>
  <div>
    <h2>Tuner:</h2>
    <h3>Frequency: {{this.frequency}} Hz</h3>
    <h3 v-show="!frequency">Please, play louder.</h3>
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
      count: 0,
      maxFrequency: 2000,
      bufferSize: 1 << 12,
      size: (1 << 12) / (1 << 10),

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
      this.analyser.minDecibels = -100
      this.analyser.maxDecibels = -10
      this.analyser.smoothingTimeConstant = 0.85
      this.mic = await navigator.mediaDevices.getUserMedia({
        audio: true
      })
      if (this.audioContext.state === 'suspended') {
        await this.audioContext.resume()
      }

      this.dataArray = new Float32Array(this.fftSize)

      this.source = this.audioContext.createMediaStreamSource(this.mic)
      this.source.connect(this.analyser)

      // this.analyser.connect(this.audioContext.destination)
    },
    test() {
      requestAnimationFrame(this.test)
      const buffer = new Float32Array(this.fftSize) // this.dataArray
      this.analyser.getFloatTimeDomainData(this.dataArray)

      // this.frequency = this.detectFrequency()
      this.frequency = this.detectFrequency() || this.frequency
    },
    detectFrequency() {
      // Perform a quick root-mean-square to see if we have enough signal
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

      // Find a range in the buffer where the values are below a given threshold.
      let r1 = 0
      let r2 = size - 1
      const threshold = 0.2

      // Walk up for r1
      for (let i = 0; i < size / 2; i++) {
        if (Math.abs(buffer[i]) < threshold) {
          r1 = i
          break
        }
      }

      // Walk down for r2
      for (let i = 1; i < size / 2; i++) {
        if (Math.abs(buffer[size - i]) < threshold) {
          r2 = size - i
          break
        }
      }

      // Trim the buffer to these ranges and update size.
      buffer = buffer.slice(r1, r2)
      size = buffer.length

      // Create a new array of the sums of offsets to do the autocorrelation
      const c = new Array(size).fill(0)
      // For each potential offset, calculate the sum of each buffer value times its offset value
      for (let i = 0; i < size; i++) {
        for (let j = 0; j < size - i; j++) {
          c[i] = c[i] + buffer[j] * buffer[j+i]
        }
      }

      // Find the last index where that value is greater than the next one (the dip)
      let d = 0
      while (c[d] > c[d+1]) {
        d++
      }

      // Iterate from that index through the end and find the maximum sum
      let maxValue = -1
      let maxIndex = -1
      for (let i = d; i < size; i++) {
        if (c[i] > maxValue) {
          maxValue = c[i]
          maxIndex = i
        }
      }

      let T0 = maxIndex

      // Not as sure about this part, don't @ me
      // From the original author:
      // interpolation is parabolic interpolation. It helps with precision. We suppose that a parabola pass through the
      // three points that comprise the peak. 'a' and 'b' are the unknowns from the linear equation system and b/(2a) is
      // the "error" in the abscissa. Well x1,x2,x3 should be y1,y2,y3 because they are the ordinates.
      const x1 = c[T0 - 1]
      const x2 = c[T0]
      const x3 = c[T0 + 1]

      const a = (x1 + x3 - 2 * x2) / 2
      const b = (x3 - x1) / 2
      if (a) {
        T0 = T0 - b / (2 * a)
      }

      return (this.audioContext.sampleRate/T0).toFixed(1)
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
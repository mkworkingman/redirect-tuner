<template>
  <div>
    <h2>Tuner:</h2>
    <audio :src="this.audioUrl" ref="track" controls></audio>
    <h3>Frequency: {{this.frequency}} Hz</h3>
  </div>
</template>

<script>
import aubio from "aubiojs";
import andante from '@/assets/andante.mp3'
import freq440hz from '@/assets/440hz.mp3'

export default {
  props: {
    fileName: String
  },
  data() {
    return {
      audioUrl: null,
      audioSource: null,
      audioContext: null,
      scriptProcessor: null,
      count: 0,
      maxFrequency: 2000,
      bufferSize: 1 << 12,
      size: (1 << 12) / (1 << 10),

      listening: true,
      keysSharp: ['C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#', 'A', 'A#', 'B'],
      keysFlat: ['C', 'Db', 'D', 'Eb', 'E', 'F', 'Gb', 'G', 'Ab', 'A', 'Bb', 'B'],
      frequency: null,
    }
  },
  methods: {
    async setup() {
      this.audioContext = new AudioContext()
      const mic = await navigator.mediaDevices.getUserMedia({
        audio: true
      })
      if (this.audioContext.state === 'suspended') {
        await this.audioContext.resume()
      }
    }
  },
  async mounted() {
    await this.setup()
  },
}
</script>

<style lang="postcss" scoped>

</style>
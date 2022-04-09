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
    run() {
      this.audioContext = new (AudioContext || webkitAudioContext)();
      this.scriptProcessor = this.audioContext.createScriptProcessor(this.bufferSize, 1, 1);
      this.audioSource = this.audioContext.createMediaElementSource(this.$refs["track"]);
      this.audioSource.connect(this.scriptProcessor);
      this.audioSource.connect(this.audioContext.destination);
      this.scriptProcessor.connect(this.audioContext.destination);

      aubio().then(({ Pitch }) => {
        const pitchDetector = new Pitch(
          "default",
          this.scriptProcessor.bufferSize,
          this.scriptProcessor.bufferSize / 8,
          this.audioContext.sampleRate
        );
        this.scriptProcessor.addEventListener("audioprocess", e => {

          const data = e.inputBuffer.getChannelData(0);
          const frequency = pitchDetector.do(data);
          if (frequency) {
            this.frequency = frequency.toFixed(1)
          }
          this.count += 1;
        });
      });
    }
  },
  mounted() {
    this.audioUrl = this.fileName === '440' ? freq440hz : andante
    this.$refs["track"].addEventListener("play", this.run);
  },
}
</script>

<style lang="postcss" scoped>

</style>
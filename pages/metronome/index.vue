<template>
  <div class="metronome">
    <div class="metronome__dashboard">
      <button
        class="metronome__btn"
        :class="{ 'metronome__btn--play' : metronomePlaying }"
        @click="metronomePlaying = !metronomePlaying"
      ></button>

    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      metronomePlaying: false
    }
  }
}
</script>

<style lang="postcss" scoped>
  .metronome {
    --width: 10px;

    &__dashboard {
      background: #8d8d8d;
      max-width: 600px;
      height: 400px;
      margin: 100px auto 0;
      padding: 40px 20px;
    }

    &__btn {
      display: block;
      margin: 40px 60px;
      width: 60px;
      height: 60px;
      border-radius: 50%;
      border: none;
      background: rgb(13, 151, 13);
      position: relative;
      outline: none;
      cursor: pointer;
      transition: all 300ms;

      &:hover,
      &:focus {
        box-shadow: 0px 5px 10px 0px rgba(0, 0, 0, 0.5);
      }

      &::before,
      &::after {
        content: '';
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translateY(-50%) translateX(calc(-50% - calc(var(--width)/2)));
        width: var(--width);
        height: 20px;
        background: rgba(255, 255, 255, 0.5);
        clip-path: polygon(0 0, 100% 25%, 100% 75%, 0 100%);
        transition: clip-path 350ms ease-in-out;
      }

      &::after {
        transform: translateY(-50%) translateX(calc(-50% + calc(var(--width)/2)));
        clip-path: polygon(0 25%, 100% 50%, 100% 50%, 0 75%);
      }

      &--play {
        background: #086d08;

        &::before {
          clip-path: polygon(0 0, 85% 0, 85% 100%, 0 100%);
        }

        &::after {
          clip-path: polygon(15% 0, 100% 0, 100% 100%, 15% 100%);
        }
      }
    }
  }
</style>
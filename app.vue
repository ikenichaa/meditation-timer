<script setup lang="ts">
import { useInterval } from "@vueuse/core";

import { useSound } from "@vueuse/sound";
import bellRing from "./assets/sounds/bell.mp3";
import relaxingSound from "./assets/sounds/30_mins_meditate.mp3";

const { play: playBell } = useSound(bellRing);
const { play: playMusic, stop: stopMusic } = useSound(relaxingSound);
const DEFAULT_MEDITATION_DURATION_IN_SECONDS = 10 * 60;
const { counter, reset, pause, resume, isActive } = useInterval(1000, {
  controls: true,
  immediate: false,
});

const resetMeditationTimer = () => {
  reset();
  pause();
};

const startMeditationTimer = () => {
  reset();
  resume();
  playBell();
  if (isMusicOn.value) {
    playMusic();
  }
};

const meditationStatus = ref<"HOME" | "INPROGRESS" | "SUCCESS">("HOME");
const meditationDurationInSeconds = ref(DEFAULT_MEDITATION_DURATION_IN_SECONDS);
const isMusicOn = ref<boolean>(false);
const onClickStartMeditation = (turnOnMusic: boolean) => {
  meditationStatus.value = "INPROGRESS";
  if (turnOnMusic) {
    isMusicOn.value = true;
  } else {
    isMusicOn.value = false;
  }

  startMeditationTimer();
};

const onClickReturnHome = () => {
  meditationStatus.value = "HOME";
  resetMeditationTimer();
};

const timer = computed(() => {
  let minutes = Math.floor(counter.value / 60);
  let extraSeconds = counter.value % 60;
  return {
    min: minutes < 10 ? "0" + minutes : minutes,
    sec: extraSeconds < 10 ? "0" + extraSeconds : extraSeconds,
  };
});

const meditateProgressBarWidth = computed(
  () => (counter.value / meditationDurationInSeconds.value) * 100
);

watch(counter, () => {
  if (counter.value === meditationDurationInSeconds.value + 1) {
    resetMeditationTimer();
    if (isMusicOn.value) {
      stopMusic();
    }

    playBell();
    meditationStatus.value = "SUCCESS";
  }
});
</script>

<template>
  <div
    v-if="meditationStatus === 'HOME'"
    class="flex w-full h-screen bg-gradient-to-r from-[#FBF7F5] to-[#F5E5DD]"
  >
    <div class="flex w-full flex-col justify-between">
      <div class="flex flex-col gap-y-2 p-10">
        <h1 class="text-3xl font-bold text-[#D28D8A]">Meditate</h1>
        <span class="text-xl text-[#938581]"
          >Path to inner peace, reduced stress, and improved focus.</span
        >
      </div>
      <div class="flex justify-center px-10">
        <img src="./assets/image/lotus.png" width="200" height="200" />
      </div>
      <div class="p-10 flex flex-col gap-y-5">
        <h1 class="text-xl text-[#6E575D]">Start meditation</h1>
        <button
          @click="onClickStartMeditation(true)"
          class="w-full bg-white rounded-md p-3"
        >
          <span class="text-[#D6938F]">10 Minutes with background music</span>
        </button>
        <button
          @click="onClickStartMeditation(false)"
          class="w-full bg-white rounded-md p-3"
        >
          <span class="text-[#D6938F]">10 Minutes in Silence</span>
        </button>
      </div>
    </div>
  </div>
  <div
    v-else-if="meditationStatus === 'INPROGRESS'"
    class="flex w-full h-screen bg-gradient-to-r from-[#FBF7F5] to-[#F5E5DD]"
  >
    <div class="w-full flex flex-col justify-between">
      <div class="flex flex-col gap-y-2 p-10">
        <h1 class="text-3xl font-bold text-[#D28D8A]">Relax</h1>
        <span class="text-xl text-[#938581]">breath deeply</span>
      </div>
      <div class="flex flex-col justify-center px-10">
        <div class="flex justify-center">
          <img src="./assets/image/meditating.png" width="400" height="400" />
        </div>
        <div class="flex flex-col justify-center p-5 gap-y-4">
          <div class="flex justify-center">
            <span class="text-[#AA6A6C] text-2xl justify-center">
              {{ timer.min }} : {{ timer.sec }}</span
            >
          </div>
          <div class="w-full bg-gray-200 rounded-full h-2.5">
            <div
              class="bg-[#AF6F70] h-2.5 rounded-full"
              :style="{ width: meditateProgressBarWidth + '%' }"
            ></div>
          </div>
          <div class="flex justify-center gap-x-2">
            <Pause v-if="isActive" @click="pause" />
            <Play v-if="!isActive" @click="resume" />
            <Reset @click="reset" />
          </div>
        </div>
      </div>
      <div class="flex justify-center p-10">
        <button
          type="button"
          @click="onClickReturnHome"
          class="text-[#D6938F] w-full bg-white rounded-md p-3 hover:bg-gradient-to-bl focus:ring-4 focus:outline-none focus:ring-red-100 dark:focus:ring-red-400"
        >
          Return to Home Page
        </button>
      </div>
    </div>
  </div>
  <div
    v-else-if="meditationStatus === 'SUCCESS'"
    class="flex w-full h-screen bg-gradient-to-r from-[#FBF7F5] to-[#F5E5DD]"
  >
    <div class="w-full flex flex-col justify-between">
      <div class="flex flex-col gap-y-2 p-10">
        <h1 class="text-3xl font-bold text-[#D28D8A]">Great Job!</h1>
        <span class="text-xl text-[#938581]"
          >You have cultivated a beautiful moment of peace. Thank you for taking
          this time for yourself.
        </span>
      </div>
      <div class="flex flex-col justify-center gap-y-2">
        <div class="flex justify-center">
          <img src="./assets/image/finish.png" width="600" height="600" />
        </div>

        <span class="text-xl text-[#938581] text-center">
          If you seek peace, be still. <br />If you seek wisdom, be silent.
          <br />
          If you seek love, be yourself
        </span>
      </div>
      <div class="flex justify-center p-10">
        <button
          type="button"
          @click="onClickReturnHome"
          class="text-[#D6938F] w-full bg-white rounded-md p-3 hover:bg-gradient-to-bl focus:ring-4 focus:outline-none focus:ring-red-100 dark:focus:ring-red-400"
        >
          Return to Home Page
        </button>
      </div>
    </div>
  </div>
</template>

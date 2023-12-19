<template>
  <div class="game-screen">
    <h3
      :key="currentQuote"
      class="game-screen__title"
    >
      Напечатайте в поле расположенную ниже цитату:
    </h3>
    <Transition
      name="fade"
      mode="out-in"
    >
      <p
        :key="currentQuote"
        class="game-screen__text"
      >
        {{ currentQuote }}
      </p>
    </Transition>
    <textarea
      ref="textareaEl"
      cols="30"
      rows="10"
      class="game-screen__textarea"
      :class="{
        '-ok': currentLetterStatus === LETTER_STATUSES.OK,
        '-error': currentLetterStatus === LETTER_STATUSES.ERROR,
      }"
      @keydown="checkKey"
    />
    <div class="game-screen__bottom-box">
      <div class="game-screen__speed-box">
        <p class="game-screen__speed-caption">
          Скорость:
        </p>
        <p class="game-screen__speed-text">
          {{ currentSpeed }} зн./мин
        </p>
      </div>
      <TimerBox :seconds-left="timerData.timesLeft" />
      <BaseButton
        class="game-screen__start-over-btn"
        @click="startOverBtnClickHandler"
      >
        Начать заново
      </BaseButton>
    </div>
  </div>
</template>

<script setup>
import {
  computed, inject, onMounted, ref,
} from 'vue';
import useDataStore from '../stores/data-store';
import useAppStore from '../stores/app-store';
import BaseButton from '../ui/BaseButton.vue';
import TimerBox from './TimerBox.vue';
import { APP_STATUSES, LETTER_STATUSES, GAME_TIMER_SECONDS } from '../../utils/constants';

const dataStore = useDataStore();
const appStore = useAppStore();

const $modals = inject(['$modals']);

const timerData = ref({
  id: null,
  timesLeft: GAME_TIMER_SECONDS,
});
const textareaEl = ref();

const currentQuote = computed(() => dataStore.currentQuote);
const currentLetterIndex = computed(() => dataStore.currentLetterIndex);
const currentSpeed = computed(() => dataStore.currentSpeed);
const startedTime = computed(() => appStore.startedTime);
const currentLetterStatus = computed(() => dataStore.currentStatus);

const calculateSpeed = () => {
  const currentTime = new Date();

  const minutesSpend = (currentTime - startedTime.value) / 1000 / 60;

  if (minutesSpend === 0) {
    dataStore.changeCurrentSpeed(0);
    return;
  }

  dataStore.changeCurrentSpeed(Math.round(currentLetterIndex.value / minutesSpend));
};
const launchTimer = () => {
  appStore.changeStartedTime(new Date());
  appStore.setTimerId(setInterval(calculateSpeed, 1000));
};
const completeGame = () => {
  appStore.changeGameStatus(APP_STATUSES.COMPLETE);
  calculateSpeed();
  appStore.clearTimer();
  clearInterval(timerData.value.id);
};
const checkKey = (event) => {
  if ($modals.active() === 'warning') {
    event.preventDefault();
    return;
  }
  if (/^[a-zA-Zа-яА-Я0-9 !"#$%&'()*+,-./:;<=>?@[\\\]^_`{|}~]$/.test(event.key)) {
    if (event.key.charCodeAt(0) >= 65 && event.key.charCodeAt(0) <= 130) {
      event.preventDefault();
      $modals.show('warning');
      return;
    }
    if (event.key.charCodeAt(0) === currentQuote.value[currentLetterIndex.value].charCodeAt(0)) {
      dataStore.changeCurrentLetterIndex(currentLetterIndex.value + 1);
      if (currentLetterIndex.value === currentQuote.value.length) {
        completeGame();
      }
      dataStore.changeCurrentLetterStatus(LETTER_STATUSES.OK);
      if (!startedTime.value) {
        launchTimer();
      }
    } else {
      event.preventDefault();
      dataStore.changeCurrentLetterStatus('error');
    }
  } else {
    event.preventDefault();
  }
};
const checkCountdown = () => {
  timerData.value.timesLeft -= 1;

  if (timerData.value.timesLeft <= 0) {
    completeGame();
  }
};
const startCountdown = () => {
  timerData.value.id = setInterval(checkCountdown, 1000);
};
const startOverBtnClickHandler = (evt) => {
  evt.currentTarget.blur();
  appStore.restartGame();
  clearInterval(timerData.value.id);
  timerData.value = {
    id: null,
    timesLeft: GAME_TIMER_SECONDS,
  };
  startCountdown();
};

onMounted(() => {
  startCountdown();
  textareaEl.value.focus();
});
</script>

<style lang="scss" scoped>
.game-screen__title {
  font-size: 1.3rem;
}
.game-screen__text {
  font-size: 1.5rem;

  span.-active {
    padding: 0.2rem;
    border-radius: 3px;
    color: white;

    &.-ok {
      background-color: green;
    }

    &.-error {
      background-color: red;
    }
  }
}

.game-screen__bottom-box {
  margin-top: 3rem;
  display: flex;
  justify-content: space-between;
  align-items: center;

  @include media-breakpoint(sm) {
    margin-top: 1.5rem;
    flex-direction: column;
    gap: 1rem;
    align-items: flex-start;
  }
}

.game-screen__speed-caption {
  font-size: 1.2rem;

  @include media-breakpoint(sm) {
    font-size: 1rem;
  }
}

.game-screen__speed-text {
  font-size: 1.4rem;
  width: rem(140);

  @include media-breakpoint(sm) {
    font-size: 1.2rem;
  }
}

.game-screen__start-over-btn {
  max-width: 15rem;

  @include media-breakpoint(sm) {
    max-width: fit-content;
  }
}

.game-screen__textarea {
  width: 100%;
  margin-top: 1rem;
  border-radius: 0.5rem;
  padding: 1rem;
  resize: none;
  height: rem(180);
  border: 2px solid black;

  &:focus {
    outline: none;
  }

  &.-ok {
    border-color: green;
  }

  &.-error {
    border-color: red;
  }
}
</style>

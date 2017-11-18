<template>
  <div class="counter">
    <div class="card-panel counter__inner">
      <!-- Header -->
      <header class="counter__header">
        <h2 contenteditable="true" v-show="heading" class="counter__heading">{{ heading }}</h2>
        <button @click="menuOpen = !menuOpen" :class="['counter__menu-toggler btn btn-flat btn-small grey-text text-darken-4']">
            <i class="material-icons">{{ menuOpen ? 'close' : 'more_vert' }}</i>
        </button>
      </header>
      <p v-show="description" contenteditable="true" class="counter__description grey-text text-darken-1">{{ description }}</p>

      <!-- - num + -->
      <div class="counter__main">
        <button :class="['btn-floating btn-large blue', {disabled: atMinRange}]" @click="decrement">
          <i class="medium material-icons">arrow_drop_down</i>
        </button>
        <span :class="['counter__value', { 'red-text': atMaxRange }]">
          <span :class="isLeadingZero(valueFormatted, index) ?
                        'grey-text text-lighten-3' : ''"
                v-for="(letter, index) in valueFormatted">{{ letter }}</span>
        </span>
        <button :class="['btn-floating btn-large red', {disabled: atMaxRange}]" @click="increment">
          <i class="medium material-icons">arrow_drop_up</i>
        </button>
      </div>

      <!-- Menu -->
      <transition @enter="enter" @leave="leave" :css="false">
        <div v-show="menuOpen" class="counter__menu hoverable z-depth-2 grey lighten-5">
          <div style="flex-grow: 0;">
            <div>Max: {{maxValue}}</div>
            <div>Step: {{step}}</div>
          </div>
          <button title="Reset" @click="reset" :class="['btn indigo', { 'disabled' : value === 0 }]">
              <i class="material-icons">exposure_zero</i>
            </button>
          <button type="submit" class="btn btn-edit green disabled">
              <i class="material-icons">create</i>
            </button>
          <button title="Remove this counter" @click="$emit('remove');" class="btn purple btn-small">
              <i class="material-icons">delete</i>
            </button>
        </div>
      </transition>
    </div>
  </div>
</template>


<script>
export default {
  props: {
    'initValue': {
      type: Number,
      default: 0
    },
    'initDigits': {
      type: Number,
      default: 1
    },
    'initHeading': {
      type: String,
      required: true
    },
    'initDescription': {
      type: String
    },
    'initStep': {
      type: Number,
      default: 1
    }
  },
  data() {
    const MAX_DIGITS = 4;
    return {
      value: this.initValue,
      newValue: null,
      digits: this.initDigits > 0 ?
        (this.initDigits < MAX_DIGITS ? this.initDigits : MAX_DIGITS) : 1,
      heading: this.initHeading,
      description: this.initDescription,
      created: new Date(),
      menuOpen: false,
      step: this.initStep
    }
  },
  computed: {
    atMinRange() {
      return this.value <= 0;
    },
    atMaxRange() {
      return this.value >= this.maxValue;
    },
    valueFormatted() {
      let length = String(this.value).length;

      if (length < this.digits) {
        return ('0'.repeat(this.digits - length) + this.value).split('');
      } else {
        return String(this.value).split('');
      }
    }
  },
  methods: {
    getMaxValue() {
      let maxValue = 0;
      for (let i = 0; i < this.digits; i++) {
        maxValue += (Math.pow(10, i) * 9)
      }
      this.maxValue = maxValue;
    },
    valInRange() {
      if (this.value < 0) this.value = 0;
      if (this.value > this.maxValue) this.value = this.maxValue;
    },
    increment() {
      this.value += this.step;
    },
    decrement() {
      this.value -= this.step;
    },
    reset() {
      this.value = 0;
      this.menuOpen = false;
    },
    setValue() {
      if (!isNaN(this.newValue) && isFinite(this.newValue) && this.newValue !== null) this.value = this.newValue;
      this.newValue = null;
    },
    isLeadingZero(arr, index) {
      // Find first normal digit and mark it's index
      let NaZ = arr.findIndex(n => n != 0);
      if (NaZ === -1) return true; // If all digits are zero return true
      if (index < NaZ) return true;
      return false;
    },
    enter(el, done) {
      TweenMax.to(el, .45, {
        y: '-100%',
        onComplete: done
      });
    },
    leave(el, done) {
      TweenMax.to(el, .45, {
        y: '0%',
        onComplete: done
      });
    }
  },
  watch: {
    value: function() {
      this.valInRange();
    }
  },
  beforeMount() {
    this.getMaxValue();
    this.valInRange();
  }
}
</script>

<style lang="scss">
.counters {
  overflow: hidden;
  padding-bottom: 6.5rem;

  @include from(medium) {
    display: flex;
    flex-wrap: wrap;
    align-items: stretch;
  }

  @include from(xlarge) {
    max-width: 1200px;
    margin: 0 auto;
  }
}

.counter {
  padding: 1rem .8rem;
  overflow: hidden;
  @include to(medium) { & + & { padding-top: 0; } }
  @include from(medium) { flex: 0 0 50%; }
  @include from(large) { flex-basis: 33.3333%; }
}

.counter__inner {
  position: relative;
  @include from(medium) {
    display: flex;
    flex-direction: column;
    height: 100%;
  }
}

.counter__header {
  display: flex;
  align-items: baseline;
}

.counter__heading {
  margin: 0 0 .35rem;
  font-size: 1.6rem;
  overflow: hidden;
  text-overflow: ellipsis;
}

.counter__menu-toggler {
  position: relative;
  margin-left: auto;
  z-index: 10;
}

.counter__description {
  margin: 0 0 .5rem;
  font-size: .9rem;
}

.counter__main {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-top: 1px solid #e0e0e0;
  padding-top: 1rem;
  margin-top: auto;
}

.counter__value {
  font-size: 1.8rem;
  span {
    transition: color .5s ease-in;
  }
}

.counter__menu {
  position: absolute;
  top: 100%;
  left: 0;
  width: 100%;
  background-color: #fff;
  z-index: 2;
  padding: 2rem;
}

</style>

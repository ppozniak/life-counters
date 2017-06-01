<template>
  <div>
    <transition-group tag="div" class="counters"
                      @enter="enterCounter"
                      @leave="leaveCounter"
                      :css="false">
      <counter v-for="(counter, index) in counters"
               :init-value="counter.value"
               :init-digits="counter.digits"
               :init-heading="counter.heading"
               :init-description="counter.description"
               :init-step="counter.step"
               :key="counter.id"
               @remove="removeCounter(counter.id)"
               />
    </transition-group>
    <div class="fixed-action-button">
      <button @click="startListening"
              :class="['btn-floating btn-large purple', {'disabled' : isListening}]"
              v-if="recognition"
              >
        <i class="material-icons medium md-light">keyboard_voice</i>
      </button>
      <button @click="addCounter()"
              class="btn-floating btn-large cyan">
        <i class="material-icons medium md-light">add</i>
      </button>
    </div>
  </div>
</template>

<script>
import Counter from './components/Counter';

export default {
  name: 'app',
  data() {
    return {
      counters: [],
      recognition: null,
      isListening: false
    }
  },
  components: {
    Counter
  },
  methods: {
    addCounter(options = {}) {
      const defaults = {
        heading: this.generateName(),
        description: this.generateName(),
        value: 0,
        digits: 3,
        step: 1,
        id: Symbol()
      }

      Object.assign(options, defaults);
      this.counters.push(options);
    },
    generateName() {
      const W1 = ['Sad', 'Poor', 'Dirty', 'Alone', 'Bitter', 'Sorrowful', 'Filthy', 'Nasty', 'Polluted', 'Messy', 'Stained', 'Greasy'],
            W2 = ['unnamed', 'forgotten', 'lost', 'abandoned', 'omitted', 'buried', 'private', 'anonymous', 'nameless', 'unknown', 'unsigned', 'unspecified', 'undefined', 'incognito'],
            r1 = W1[Math.floor(Math.random() * W1.length)],
            r2 = W2[Math.floor(Math.random() * W2.length)];
      return `${r1} ${r2} counter`;
    },
    removeCounter(index) {
      const counterId = this.counters.findIndex(counter => counter.id === index);
      this.counters.splice(counterId, 1);
    },
    enterCounter(el, done) {
      TweenMax.to(window, .8, {
        scrollTo: el
      })
      TweenMax.from(el, .4, {
        x: '-100%',
        opacity: 0,
        onComplete: done
      });
    },
    leaveCounter(el, done) {
      TweenMax.to(el, .8, {
        height: '0px',
        x: '100%',
        opacity: '0',
        padding: '0px',
        margin: '0px',
        border: '0',
        flexBasis: '0%',
        onComplete: done
      });
    },
    startListening() {
      console.log('Listening!');
      this.isListening = true;
      this.recognition.start();
    },
    listening(e) {
      const transcript = Array.from(e.results)
                              .map(result => result[0])
                              .map(result => result.transcript)
                              .join('');
      console.log(transcript);
      if(/\b(add|new|insert|put)\b/.test(transcript)) {
        this.recognition.abort();
        this.voiceCommand(this.addCounter);
      }
    },
    listeningEnd(e) {
      this.isListening = false;
      console.log('Done!');
    },
    voiceCommand: _.debounce(function(cmdFunction) {
      console.log('executing!');
      cmdFunction();
    }, 100)
  },
  mounted() {
    // Add initially 3 counters
    this.addCounter();
    setTimeout(() => {
      this.addCounter();
    }, 500);
    setTimeout(() => {
      this.addCounter();
    }, 1000);
    this.recognition = speechRecognition(this.listening, this.listeningEnd);
  }
}

function speechRecognition(listening, listeningEnd) {
  try {
    window.SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    window.SpeechGrammarList = window.SpeechGrammarList || window.webkitSpeechGrammarList;
    const recognition = new SpeechRecognition();
    const grammar = `
      #JSGF V1.0;
      grammar actions;
      public <action> = <kind> (put | insert | add) [a] [new] counter;
      <kind> = (please | kindly | could you)*;
      `
    const grammarList = new SpeechGrammarList();
    grammarList.addFromString(grammar);
    recognition.grammar = grammarList;
    recognition.interimResults = true;
    recognition.lang = 'EN-us';
    recognition.addEventListener('result', listening);
    recognition.addEventListener('end', listeningEnd);
    return recognition;
  } catch(e) {
    console.error(e)
    console.log('Voice recognition is not supported!');
    return false;
  }
}
</script>

<style lang="scss">
button, input, textarea, form, fieldset {
-webkit-appearance: none;
-moz-appearance: none;
border: none;
color: inherit;
padding: 0;
margin: 0;
}


.btn-small {
padding: 0 .6rem;
}
/* Rules for sizing the icon. */
.material-icons {
user-select: none;
}
.material-icons.small { font-size: 18px; }
.material-icons.medium { font-size: 36px; }
.material-icons.large { font-size: 48px; }

/* Rules for using icons as black on a light background. */
.material-icons.md-dark { color: rgba(0, 0, 0, 0.54); }
.material-icons.md-dark.md-inactive { color: rgba(0, 0, 0, 0.26); }

/* Rules for using icons as white on a dark background. */
.material-icons.md-light { color: rgba(255, 255, 255, 1); }
.material-icons.md-light.md-inactive { color: rgba(255, 255, 255, 0.3); }

.card-panel {
margin: 0;
}

.fixed-action-button {
position: fixed;
z-index: 10;
bottom: .6rem;
right: .6rem;
}
</style>

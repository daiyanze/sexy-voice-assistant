<template>
  <div class="mx-auto w-full">
    <pre class="message-box" v-html="context" ref="messageBox"></pre>
    <button class="button-control" @mousedown="listen(true)" @mouseup="listen(false)">Hold to Listen</button>
    <button class="button-control" @click="reset">Reset</button>
    <select class="voice-options" v-model="activeVoiceIdx">
      <option :key="key" :value="key" :selected="key === activeVoiceIdx" v-for="(val, key) in voices">
        {{ val.name }} ({{ val.lang }})
      </option>
    </select>
  </div>
</template>
<script>
import { onMounted, reactive, toRefs, ref } from '@vue/composition-api'

export default {
  setup (_, { root }) {
    const getVoices = async (window) => {
      let id, res

      await new Promise(resolve => {
        id = setInterval(() => {
          res = window.speechSynthesis.getVoices()
          if (res.length !== 0) {
            resolve(res)
            clearInterval(id)
          }
        }, 10)
      })

      return res
    }

    const process = (msg, voice) => {
      const content = msg.toLowerCase()

      if (/^google search/g.test(content)) {
        const url = `https://google.com/search?q=${msg.replace('Google search ', '')}`
        window.open(url, '_blank')
        return `Base on your query, I found some search results on Google`
      } else if (/your name/g.test(content)) {
        return `My name is ${voice.name}.`
      } else if (/(hello|hi)/g.test(content)) {
        return 'Hi! Nice to meet you!'
      } else if (/love you/g.test(content)) {
        return 'I love you too. And I\'ll love you forever!'
      } else if (/(naked|nude|tits|breast|butt|ass|shit|dick|pussy|asshole)/g.test(content)) {
        return 'I know I love you but can you show some politeness in front of a lady?'
      }

      return 'Sorry, my sweetheart. I don\'t understand. Could you try something else?'
    }

    const state = reactive({
      name: '',
      context: 'Sorry, your browser doesn\'t support SpeechRecognition. Please use Chrome / Edge79+ instead.',
      voices: [],
      activeVoiceIdx: 17
    })

    const messageBox = ref(null)

    const SpeechRecognition = window.webkitSpeechRecognition && new window.webkitSpeechRecognition()
    SpeechRecognition && (SpeechRecognition.interimResults = true)

    const listen = start => {
      if (!SpeechRecognition) return

      if (start) {
        SpeechRecognition.start()
      } else {
        SpeechRecognition.stop()
      }
    }

    const reset = () => {
      if (!SpeechRecognition) return

      state.context = '<b>' + state.name + '</b>' + ': Hi, my hero!'
      speechSynthesis.cancel()
    }

    onMounted(async () => {

      if (!window.webkitSpeechRecognition) return

      state.voices = await getVoices(window)
      state.name = state.voices[state.activeVoiceIdx].name
      state.context = '<b>' + state.name + '</b>' + ': Hi, my hero!'

      const utterThis = new window.SpeechSynthesisUtterance()

      const speechSpeaker = (text, voice) => {
        utterThis.text = text
        utterThis.voice = voice
        utterThis.pitch = 1.4
        utterThis.lang = voice.lang
        window.speechSynthesis.speak(utterThis)
      }

      SpeechRecognition.onresult = event => {
        const result = event.results[event.results.length - 1]

        if (result.isFinal) {
          state.context += '\n<b>Me:</b> ' + result[0].transcript

          const voice = state.voices[state.activeVoiceIdx]
          const reply = process(result[0].transcript, voice)
          root.$nextTick(() => {
            messageBox.value.scrollTop = messageBox.value.scrollHeight
          })

          setTimeout(() => {
            state.context += '\n<b>' + voice.name + '</b>: ' + reply

            root.$nextTick(() => {
              messageBox.value.scrollTop = messageBox.value.scrollHeight
            })

            speechSpeaker(reply, voice)
          }, 600)
        }
      }

    })

    return {
      ...toRefs(state),
      listen,
      reset,
      messageBox
    }
  }
}
</script>
<style lang="postcss" scoped>
.button-control {
  @apply font-semibold;
  @apply px-4 py-2;
  @apply mr-2 mb-2;
  @apply bg-gray-600;
  @apply border-none;
  @apply rounded-sm;
  @apply shadow-lg;
  @apply text-white;

  &:focus {
    @apply outline-none;
  }

  &:active {
    @apply bg-gray-700;
    @apply shadow-none;
  }
}

.message-box {
  @apply relative;
  @apply h-48;
  @apply overflow-y-scroll;
  @apply rounded-sm;
  @apply border border-solid border-gray-300;
  @apply p-2;
  @apply mb-2;
  @apply leading-6;
}

.voice-options {
  @apply font-semibold;
  @apply bg-gray-100;
  @apply px-2 py-1;
  @apply mr-2 mb-2;
  @apply rounded-sm;
  @apply border-2 border-solid border-gray-100;
  @apply shadow-md;

  &:focus {
    @apply outline-none;
  }
}
</style>
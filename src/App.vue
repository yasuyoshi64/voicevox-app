<script setup>
import { ref } from 'vue'
import axios from 'axios'

const text = ref('')
const audio_data = ref(null)
const audio = ref(null)

// VOICEVOXにテキストの音声データをリクエスト
async function request() {
  await axios
    .post(
      encodeURI('http://localhost:50021/audio_query?text=' + text.value + '&speaker=1'), 
      null, 
      { headers: { 'Content-Type': 'application/json; charset=UTF-8'} }
    )
    .then(async (response) => {
      await axios
        .post(
          encodeURI('http://localhost:50021/synthesis?speaker=1&enable_interrogative_upspeak=true'),
          response.data,
          { headers: { 'Content-Type': 'application/json; charset=UTF-8'}, responseType: 'arraybuffer' }
        )
        .then((response2) => {
          // 音声データを<audio>のsrcに投入
          var blob = new Blob([response2.data],  { type: 'audio/wav' })
          audio_data.value = URL.createObjectURL(blob)
        })
    })
 }

 // 再生準備完了
 function canplay() {
    audio.value.play()    // 再生させます。
 }

</script>

<template>
  <div>
    <p>テキスト <input v-model="text" size="35" /> <button @click="request">再生</button></p>
    <audio controls :src="audio_data" ref="audio" type="audio/wav" @canplay="canplay" />
  </div>
</template>

<template>
  <div class="player-control" >
    <div class="control-item" @click="rotateDeg = rotateDeg + 90">
      旋转:
      <n-icon size="large" color="#306eff" >
        <rotate-2></rotate-2>
      </n-icon>
    </div>
    <div class="control-item">
      缩放:
      <n-space vertical>
        <n-slider v-model:value="scale" :step="10" />
      </n-space>
    </div>
    <div class="control-item">
      评价:
      <n-space align="center">
        <n-rate size="large" @update:value="saveRate" v-model:value="rate" />
      </n-space>
    </div>
    <div class="control-item" v-if="lastPlayedTime > 0">
      <n-button size="small" @click="continueLast">继续上次播放: {{ parseInt(lastPlayedTime / videoInfo.params.duration * 100) + '%' }}</n-button>
    </div>
  </div>
    <video ref="playerRef" :data-poster="videoInfo.thumbnail_link" v-if="videoInfo.mime_type.indexOf('video') !== -1" >
      <template v-for="(item, key) in videoInfo.medias">
        <source :src="item?.link.url" :type="videoInfo.mime_type === 'video/x-matroska' ? 'video/webm' :videoInfo.mime_type" :size="list[item.resolution_name]" />
      </template>
    </video>
    <audio ref="playerRef" v-else-if="videoInfo.mime_type.indexOf('audio') !== -1">
        <source :src="videoInfo.web_content_link"  />
    </audio>
</template>
<script setup lang="ts">
import { computed, ref } from '@vue/reactivity';
import { onMounted, watch } from '@vue/runtime-core';
import { NxButton, NButton, NSlider, NSpace, NRate, NIcon } from 'naive-ui'
import { Rotate2, Resize } from '@vicons/tabler'
import Plyr from 'plyr'
import { throttle } from '../utils'
import 'plyr/dist/plyr.css'

  

  const rotateDeg = ref(0);
  const scale = ref(50);
  watch([rotateDeg, scale], () => {
    document.querySelector('.plyr__video-wrapper video').style.transform = `rotate(${rotateDeg.value}deg) scale(${scale.value / 100 * 2})`;
  })
  const props = defineProps<{
    video: {
      [key:string]:any
    }
  }>()
  const videoInfo = computed(() => {
    return props.video
  })
  console.log('[videoInfo]', videoInfo);
  const rate = ref((localStorage.getItem(`VideoRate_${videoInfo.value.name}`) || 0) - 0);
  const lastPlayedTime = ref((localStorage.getItem(`VideoLastPlayedTime_${videoInfo.value.name}`) || 0) - 0);
  const saveRate = (value:number) => {
    window.localStorage.setItem(`VideoRate_${videoInfo.value.name}`, value + '')
  }
  

  const list:{
    [key:string]:number
  } = {
    '4K': 2160,
    '1080P': 1080,
    '720P': 720,
    '480P': 480
  }
  const playerRef = ref()
  const player = ref()
  const initPlayer = () => {
    player.value = new Plyr(playerRef.value, {
      autoplay: true,
      // debug: true
    })
    player.value.on('timeupdate', throttle(() => {
      localStorage.setItem(`VideoLastPlayedTime_${videoInfo.value.name}`, player.value.currentTime)
    } , 3000))
    window.videoPlayer = player;
  }

  const continueLast = () => {
    player.value.currentTime = lastPlayedTime.value - 0;
  }

  onMounted(initPlayer)
</script>

<style>
.player-control{
  position: relative;
  top: -14px;
  font-size: 18px;
  display: flex;
}
.player-control .control-item{
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 0 10px;
}
.control-item .n-space{
  flex: 1;
  padding: 0 10px;
}
.plyr--video {
  height: 100%;
}
.plyr--audio {
  margin-top: 320px;
}
.plyr video {
  object-fit: contain;
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  width: 100%;
  height: 100%;
}

</style>
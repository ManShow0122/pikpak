<template>
  <div class="rotate-control" @click="rotateDeg = rotateDeg + 90">旋转</div>
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
import Plyr from 'plyr'
import 'plyr/dist/plyr.css'

  const rotateDeg = ref(0);
  watch(rotateDeg, () => {
    document.querySelector('.plyr__video-wrapper video').style.transform = `rotate(${rotateDeg.value}deg)`;
  })
  const props = defineProps<{
    video: {
      [key:string]:any
    }
  }>()
  const videoInfo = computed(() => {
    return props.video
  })
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
    window.videoPlayer = player;
  }
  onMounted(initPlayer)
</script>

<style>
.rotate-control{
  position: relative;
  top: -14px;
  font-size: 18px;
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
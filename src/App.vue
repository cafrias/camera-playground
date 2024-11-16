<script setup lang="ts">
import { useQuery } from '@tanstack/vue-query'

// Following article:
// https://web.dev/articles/getusermedia-intro#setting_media_constraints_resolution_height_width

const { isPending, isError, data: cameraStream, error, refetch } = useQuery<MediaStream, Error>({
  queryKey: ['cameraStream'],
  queryFn: async () => {
    if (!navigator.mediaDevices) {
      // TODO: create custom error type
      throw new Error('No media devices available')
    }

    const devices = await navigator.mediaDevices.enumerateDevices()
    const camera = devices.find((device) => device.kind === "videoinput")
    if (!camera) {
      // TODO: create custom error type
      throw new Error('No camera')
    }

    return navigator.mediaDevices.getUserMedia({
      video: {
        deviceId: camera.deviceId
      }
    })
  }
})

function stop() {
  const {value} = cameraStream;
  if (!value) {
    return
  }
  value.getVideoTracks().forEach(track => track.stop());
}

function start() {
  refetch();
}

</script>

<template>
  <main>
    <span v-if="isPending">Loading...</span>
    <span v-else-if="isError">Error: {{ error }}</span>
    <template v-else>
      <video autoplay :srcObject="cameraStream"></video>
      <button @click="stop">Stop</button>
      <button @click="start">Start</button>
    </template>
  </main>
</template>

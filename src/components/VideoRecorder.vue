<template>
  <div>
    <video ref="video" width="500" height="500" autoplay></video>
    <video ref="videoPreview" width="200" height="200" autoplay></video> 
    <div>
      <button @click="onStartRecording()">start</button>
      <button @click="stopRecording()">stop</button>
      <button>
        <a :href="downloadLink" download="RecordedVideo.webm">download</a>
      </button>
    </div>
  </div>
</template>

<script>
export default {
  async mounted() {
    this.browserHasSupport = !!(
      navigator.mediaDevices && navigator.mediaDevices.getUserMedia
    );

    await this.getCameraImages();
    this.getVideoPermissions();
  },
  data() {
    return {
      browserHasSupport: false,
      shouldStop: false,
      stopped: false,
      downloadLink: '',
      dataArray: [{ name: 'Cristian' }, { name: 'Marcelo' }],
      selectedValue: '',
    };
  },
  methods: {
    getVideoPermissions() {

      // check camera is allowed.
      navigator.permissions.query({name: 'camera'})
        .then(data => {
          console.log('mudou', data);
          // data.onchange = (newValue) => {console.log(newValue)}
        })
    },
    async getCameraImages() {

      const uniqueArrayObj = (array) => 
        array.filter((item, idx) => array.findIndex(x => x.groupId === item.groupId) === idx);

      let audioDevices = (await navigator.mediaDevices.enumerateDevices()).filter(x => x.kind === 'audioinput');
      let videoDevices = (await navigator.mediaDevices.enumerateDevices()).filter(x => x.kind === 'videoinput');

      audioDevices = uniqueArrayObj(audioDevices);
      videoDevices = uniqueArrayObj(videoDevices);

      console.log({
        Audio: audioDevices,
        Video: videoDevices
      });

      navigator.mediaDevices.getUserMedia({
        video: true,
        audio: true,
      })
      .then(stream => {
        this.preview.srcObject = stream
        this.preview.play();
      });
    },
    onStartRecording() {
      this.shouldStop = false;
      const optionsRecorder = {
        mimeType: 'video/webm',
      };
      const recordedChunks = [];
      const mediaRecorder = new MediaRecorder(this.preview.srcObject, optionsRecorder)

      mediaRecorder.addEventListener('dataavailable', (e) => {
        
        if (e.data.size > 0) {
          console.log('recording...');
          recordedChunks.push(e.data);
        }

        if (this.shouldStop && !this.stopped) {
          mediaRecorder.stop();
          this.stopped = true;
        }
      })

      mediaRecorder.addEventListener('stop', () => {
        this.downloadLink = URL.createObjectURL(new Blob(recordedChunks));
        this.videoReproduce.src = this.downloadLink;
      });

      mediaRecorder.start(1000);
      console.log(mediaRecorder);
    },
    stopRecording() {
      this.shouldStop = true;
    },
    playVideoFromBlob(){
      // const mediaSrc = new MediaSource();
      // const mediaSrcUrl = url.createObjectURL(mediaSrc);
      // const video = this.videoReproduce;
      // video.srcObject = mediaSrcUrl;
    }
  },
  computed: {
    preview() {
      return this.$refs.video;
    },
    videoReproduce() {
      return this.$refs.videoPreview;
    }
  },
  watch: {
    selectedValue(newValue) {
      console.log(newValue);
    }
  }
};
</script>

<style lang="scss" scoped>
</style>
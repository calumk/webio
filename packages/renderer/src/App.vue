<script setup lang="ts">

  import { ref, reactive, onMounted, onUnmounted } from 'vue'
  import Hls from 'hls.js';

  import { stations } from './stations.js'

  let hls = new Hls();

  const audioplayer : HTMLAudioElement = ref(null);

  let obj = reactive({
    freq : []
  })


  // 

  let newStation = (item) => {
    // audioplayer.value.pause();
    console.log("Pausing Audio");
    audioplayer.value.pause();
    console.log("Changing station ", item.name)
    hls.loadSource(item.url);
    // audioplayer.value.play();
  }



  onMounted(() => {


    newStation(stations[0])

    hls.attachMedia(audioplayer.value);


    hls.on(Hls.Events.MANIFEST_PARSED, function () {
      console.log("MANIFEST_PARSED")
      audioplayer.value.play();
    });

    hls.on(Hls.Events.ERROR, function (event, data) {
      if (data.fatal) {
        switch (data.type) {
          case Hls.ErrorTypes.NETWORK_ERROR:
            // try to recover network error
            console.log('fatal network error encountered, try to recover');
            hls.startLoad();
            break;
          case Hls.ErrorTypes.MEDIA_ERROR:
            console.log('fatal media error encountered, try to recover');
            hls.recoverMediaError();
            break;
          default:
            // cannot recover
            console.log("DIIIIIEEE")
            hls.destroy();
            break;
        }
      }
    });


    let ctx = new AudioContext();
    let analyser = ctx.createAnalyser();
    let audioSrc = ctx.createMediaElementSource(audioplayer.value);

    audioSrc.connect(analyser);
    analyser.connect(ctx.destination);
    analyser.fftSize = 32;
    obj.freq = new Uint8Array(analyser.frequencyBinCount);

   setInterval(()=>{
    var array = new Uint8Array(analyser.frequencyBinCount);
    analyser.getByteFrequencyData(array);
    obj.freq = array
   }, 100)

  });


</script>




<template>
  <audio ref="audioplayer" width="300" height="100" controls></audio>
  <div class="window">

 <el-scrollbar height="300px">
    <p v-for="item in stations" :key="item" class="scrollbar-demo-item" @click="newStation(item)">{{ item.name }}</p>
  </el-scrollbar>

  </div>
  <div class="bars">
    <div v-for="bar in obj.freq" class="bar" :style="'height:' + bar/8 + 'px;'"></div>
  </div>
</template>





<style>


.scrollbar-demo-item {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 40px;
  margin: 10px;
  text-align: center;
  border-radius: 4px;
  background: #222;
  color: #fff;
}


.window{
  background-color: #fff;
  padding-bottom:10px;
}
.bars{
  width: 100%;
  display: flex;
  /* display: none; */
  flex-direction: row;
  justify-content: center;
  height:100px;
}

.bar{
    display: inline-block;
    width:20px;
    /* margin:1px; */
    background-color: #fff;
    height:10px;
}

audio { display:none;}


#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin:0px;
  padding:0px;
  /* margin-top: 60px; */
}

.logo-box {
  display: flex;
  width: 100%;
  justify-content: center;
}

.logo-box span {
  width: 74px;
}

.static-public {
  display: flex;
  align-items: center;
  justify-content: center;
}

.static-public code {
  background-color: #eee;
  padding: 2px 4px;
  margin: 0 4px;
  border-radius: 4px;
  color: #304455;
}
</style>

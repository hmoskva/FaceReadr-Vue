<template>
  <div id="app">
    <div>
      Face Readr
    </div>
    <video ref="video" width="500" height="500" autoplay muted></video>
  </div>
</template>

<script>
import * as faceApi from "face-api.js";

export default {
  methods: {
    start() {
      navigator.getUserMedia(
        { video: {} },
        stream => {
          const { video } = this.$refs;
          video.srcObject = stream;
          video.addEventListener("play", () => {
            const canvas = faceApi.createCanvasFromMedia(video);
            document.body.append(canvas);
            const displaySize = { width: video.width, height: video.height };
            faceApi.matchDimensions(canvas, displaySize);
            setInterval(async () => {
              const detections = await faceApi
                .detectAllFaces(video, new faceApi.TinyFaceDetectorOptions())
                .withFaceLandmarks()
                .withFaceExpressions();
              console.log(detections); // [{expressions: {angry, disgusted, fearful, happy, neutral, sad, surprised}}]
              const resizedDetections = faceApi.resizeResults(
                detections,
                displaySize
              );
              canvas
                .getContext("2d")
                .clearRect(0, 0, canvas.width, canvas.height);
              faceApi.draw.drawDetections(canvas, resizedDetections);
              faceApi.draw.drawFaceLandmarks(canvas, resizedDetections);
              faceApi.draw.drawFaceExpressions(canvas, resizedDetections);
            }, 100);
          });
        },
        err => console.error(err)
      );
    }
  },
  mounted() {
    Promise.all([
      faceApi.nets.tinyFaceDetector.loadFromUri("/models"),
      faceApi.nets.faceLandmark68Net.loadFromUri("/models"),
      faceApi.nets.faceRecognitionNet.loadFromUri("/models"),
      faceApi.nets.faceExpressionNet.loadFromUri("/models")
    ])
      .then(this.start)
      .catch(err => console.log(err));
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

body {
  margin: 0;
  padding: 0;
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
}

#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}

canvas {
  position: absolute;
}
</style>

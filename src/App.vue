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
    data() {
      return {
        interval: ""
      };
    },
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
            this.interval = setInterval(async () => {
              const detection = await faceApi
                .detectSingleFace(
                  video,
                  new faceApi.TinyFaceDetectorOptions({ scoreThreshold: 0.01 })
                )
                .withFaceLandmarks()
                .withFaceExpressions();
              // console.log(detection, "det");
              this.showToast(detection); // [{expressions: {angry, disgusted, fearful, happy, neutral, sad, surprised}}]
              // const resizedDetection = faceApi.resizeResults(
              //   detection,
              //   displaySize
              // );
              // canvas
              //   .getContext("2d")
              //   .clearRect(0, 0, canvas.width, canvas.height);
              // faceApi.draw.drawDetections(canvas, resizedDetection);
              // faceApi.draw.drawFaceLandmarks(canvas, resizedDetection);
              // faceApi.draw.drawFaceExpressions(canvas, resizedDetection);
            }, 3000);
          });
        },
        err => console.error(err)
      );
    },
    showToast(detection) {
      //TODO: Get max expression score
      if (detection.expressions) {
        const maxScore = Math.max.apply(
          null,
          Object.values(detection.expressions)
        );

        const expression = Object.keys(detection.expressions).find(
          key => detection.expressions[key] == maxScore
        );
        console.log(expression, "expr");
        this.$toasted.info(`You are ${expression}`);
        // for (let expression in detection.expressions) {
        //   if (detection.expressions[expression]) {
        //     console.log(`You are ${expression}`);
        //     // this.$toasted.show(`You are ${expression}`);
        //   }
        // }
      }
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
  },
  beforeDestroy() {
    clearInterval(this.intercal);
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

<template>
  <div>
    <Blur v-if="isBlur" />
    <div class="container">
      <div class="title">Moving: {{isBlur}}</div>
      <div class="subtitle">Alerts user to not use the device on motion detection</div>
      <div>acceleration: {{ getFixedNumber(acceleration.average) }}</div>
      <div>xyzAccelerationSum: {{ getFixedNumber(xyzAccelerationSum) }}</div>
      <div>isIphone: {{ isIphone }}</div>
      <button v-if="isIphone" @click="requestDeviceMotionPermission">Start</button>
    </div>
  </div>
</template>

<script>
import Blur from "../components/Blur.vue";

export default {
  components: {
    Blur
  },
  data() {
    return {
      acceleration: "",
      isBlur: false,
      acceleration: {
        average: 0,
        n: 0
      },
      xyzAccelerationSum: 0,
      isIphone: false
    };
  },
  computed: {
    accelerationData() {
      return `${this.getFixedNumber(this.acceleration.average)}`;
    }
  },
  methods: {
    requestDeviceMotionPermission() {
      DeviceMotionEvent.requestPermission()
        .then(permissionState => {
          if (permissionState === "granted") {
            window.addEventListener("devicemotion", this.processMotion);
          } else {
            alert("This app requires DeviceMotion permission.");
          }
        })
        .catch(console.error);
    },
    detectIphone() {
      this.isIphone =
        DeviceMotionEvent &&
        typeof DeviceMotionEvent.requestPermission === "function";
    },
    getFixedNumber(x) {
      return Number.parseFloat(x).toFixed(2);
    },
    limitMaximumAcceleration() {
      if (this.acceleration.average > 4) {
        this.acceleration.average = 4;
      }
    },
    isFilteredAccelerating(a) {
      if (!a) return;
      this.xyzAccelerationSum = Math.abs(a.x) + Math.abs(a.y) + Math.abs(a.z);

      if (this.acceleration.n === 0) {
        this.acceleration.average = this.xyzAccelerationSum;
        this.acceleration.n = 1;
      } else {
        this.acceleration.average =
          (this.acceleration.average * this.acceleration.n +
            this.xyzAccelerationSum) /
          (this.acceleration.n + 1);
        this.limitMaximumAcceleration();
      }

      if (this.acceleration.n < 100) this.acceleration.n++;

      return this.acceleration.n >= 100 && this.acceleration.average > 1;
    },
    attachMotionDetection() {
      window.addEventListener("devicemotion", this.processMotion);
    },
    processMotion(event) {
      this.isBlur = this.isFilteredAccelerating(event.acceleration);
    }
  },
  mounted: function() {
    this.detectIphone();
    if (!this.isIphone) this.attachMotionDetection(); // can't automatically start motion detection on iPhone :(
  },
  destroyed() {
    window.removeEventListener("devicemotion", this.processMotion);
  }
};
</script>

<style>
.container {
  padding: 2rem;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  flex-direction: column;
}

.title {
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont,
    "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  font-size: 4rem;
}

.subtitle {
  font-size: 2rem;
}
</style>

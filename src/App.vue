<template>
  <div id="app">
    <div id="desktop">
      <div id="canvas-container"></div>
    </div>
    <button @click="clickety()">Oi!</button>
    <div id="toolbar">
      <div>
        <template v-for="(value, format) in formats">
          <input
            type="radio"
            :id="`format-${format}`"
            :key="`format-radio-${format}`"
            :value="format"
            v-model="socialMedia"
            name="social_media_format"
          />
          <label :key="`format-label-${format}`" :for="`format-${format}`"
            >{{ value }}
          </label>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
import P5 from "p5";
import json from "@/assets/config.json";

export default {
  name: "App",
  data() {
    return {
      sketch: null,
      p5instance: null,
      canvas: null,
      config: json,
      socialMedia: "facebook_feed",
      formats: {
        facebook_feed: "Facebook",
        twitter_feed: "Twitter",
        instagram_feed: "Timeline Instagram",
        instagram_stories: "Stories Instagram",
      },
      editions: ["ONHB13", "ONHB12", "Pré-2020", "Neutro"],
    };
  },
  watch: {
    socialMedia: function () {
      this.p5instance.updateFormat();
    },
  },
  components: {},
  methods: {
    clickety: function () {
      this.p5instance.redraw();
      this.p5instance.saveImg();
    },
  },
  mounted() {
    this.sketch = (s) => {
      let mainFont, auxFont, auxFont2, logo;

      s.preload = () => {
        mainFont = s.loadFont("./fonts/CooperHewitt-Semibold.otf");
        auxFont = s.loadFont("./fonts/CooperHewitt-Book.otf");
        auxFont2 = s.loadFont("./fonts/CooperHewitt-MediumItalic.otf");
        logo = s.loadImage("./img/logo.png");
      };

      s.setup = () => {
        this.canvas = s.createCanvas(
          this.config.formats[this.socialMedia].w,
          this.config.formats[this.socialMedia].h
        );
        this.canvas.parent("canvas-container");
        s.updateZoom();
        console.log(`${mainFont}/${auxFont}/${auxFont2}/${logo}`);
      };

      s.updateZoom = () => {
        // debugger; // eslint-disable-line no-debugger
        let availableHSpace = document.getElementById("desktop").clientHeight,
          ratioH = availableHSpace / this.config.formats[this.socialMedia].h,
          availableWSpace = document.getElementById("desktop").clientWidth,
          ratioW = availableWSpace / this.config.formats[this.socialMedia].w;

        if (ratioW < 1 || ratioH < 1) {
          let scale = Math.min(ratioW, ratioH);
          this.canvas.elt.style.transform = `scale(${scale})`;
        } else {
          this.canvas.elt.style.transform = `scale(1)`;
        }
      };

      s.updateFormat = () => {
        let dimensions = this.config.formats[this.socialMedia];
        s.resizeCanvas(dimensions.w, dimensions.h);
        s.updateZoom();
      };
      s.updatePattern = () => {};
      s.updateTextVars = () => {};

      s.saveImg = () => {
        let now = new Date();
        let clock = now.getHours() + "·" + now.getMinutes();
        let day = now.toJSON().slice(0, 10);
        let fn = `post-${day}-${clock}.jpg`;
        s.saveCanvas(this.canvas, fn, "jpg");
      };

      s.draw = () => {
        s.background("#ff00ff");
        s.noLoop();
      };

      // create methods:
      // s.yourMethod = (x, y) => {
      //   // your method
      // };
    };

    this.p5instance = new P5(this.sketch, this.canvas);
  },
};
</script>

<style lang="scss">
$p: 3rem;

body {
  margin: 0;
}

#toolbar {
  position: fixed;
  z-index: 1;

  left: 0;
  top: 0;
  bottom: 0;
  width: $p;
  color: white;
  padding: 0.25rem;

  background: #333;

  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

#desktop {
  // overflow-y: scroll;
  position: fixed;
  left: $p;
  top: 0;
  bottom: 0;
  right: 0;
  background: black;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>

<template>
  <div id="app">
    <div id="desktop">
      <div id="canvas-container"></div>
    </div>
    <button @click="clickety()">Oi!</button>
    <div id="toolbar">
      <div>
        <template v-for="(value, formatKey) in formats">
          <input
            type="radio"
            :id="`format-${formatKey}`"
            :key="`format-radio-${formatKey}`"
            :value="formatKey"
            v-model="format"
            name="social_media_format"
          />
          <label :key="`format-label-${formatKey}`" :for="`format-${formatKey}`"
            >{{ value }}
          </label>
        </template>
      </div>
      <div>
        <template v-for="(pattern, key) in patterns">
          <input
            type="checkbox"
            :id="`pattern-${key}`"
            v-model="pattern.checked"
            :key="`pattern-${key}`"
            @change="p5instance.draw()"
          />
          <label :key="`pattern-label-${key}`" :for="`pattern-${key}`"
            >{{ pattern.name }}
          </label>
        </template>
      </div>
      <div id="img-input-container"></div>
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
      configRef: json,
      tint: 1.0,
      patterns: {
        circle: { name: "Circular", checked: false },
        border: { name: "Borda", checked: false },
        full: { name: "Pleno", checked: false },
        one_third: { name: "⅓", checked: false },
        two_thirds: { name: "⅔", checked: false },
      },
      format: "facebook_feed",
      formats: {
        facebook_feed: "Facebook",
        twitter_feed: "Twitter",
        instagram_feed: "Timeline Instagram",
        instagram_stories: "Stories Instagram",
      },
      edition: "ONHB12",
      editions: ["ONHB12", "Pré-2020", "Neutro"],
    };
  },
  watch: {
    format: function () {
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
      let mainFont,
        auxFont,
        auxFont2,
        logo,
        userImg = null,
        originalUserImg = null,
        loadedPatterns = {};

      s.preload = () => {
        mainFont = s.loadFont("./fonts/CooperHewitt-Semibold.otf");
        auxFont = s.loadFont("./fonts/CooperHewitt-Book.otf");
        auxFont2 = s.loadFont("./fonts/CooperHewitt-MediumItalic.otf");
        logo = s.loadImage("./img/logo.png");
      };

      s.setup = () => {
        this.canvas = s.createCanvas(
          this.configRef.formats[this.format].w,
          this.configRef.formats[this.format].h
        );

        let imgInput = s.createFileInput(s.handleUpload);
        imgInput.id("img-upload");
        let container = s.select("#img-input-container");
        container.child(imgInput);

        this.canvas.parent("canvas-container");
        s.updateZoom();
        console.log(`${mainFont}/${auxFont}/${auxFont2}/${logo}`);
      };

      s.updateZoom = () => {
        // debugger; // eslint-disable-line no-debugger
        let availableHSpace = document.getElementById("desktop").clientHeight,
          ratioH = availableHSpace / this.configRef.formats[this.format].h,
          availableWSpace = document.getElementById("desktop").clientWidth,
          ratioW = availableWSpace / this.configRef.formats[this.format].w;

        if (ratioW < 1 || ratioH < 1) {
          let scale = Math.min(ratioW, ratioH);
          this.canvas.elt.style.transform = `scale(${scale})`;
        } else {
          this.canvas.elt.style.transform = `scale(1)`;
        }
      };

      s.windowResized = () => {
        s.updateZoom();
      };

      s.updateFormat = () => {
        let dimensions = this.configRef.formats[this.format];
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

      s.drawImages = () => {
        if (userImg) {
          s.tint(203, 0, 114);
          let ratio,
            rCanv = s.width / s.height,
            rImg = userImg.width / userImg.height;

          if (rCanv >= rImg) {
            ratio = s.width / userImg.width;
          } else {
            ratio = s.height / userImg.height;
          }
          let newW = userImg.width * ratio,
            newH = userImg.height * ratio,
            x = s.width / 2 - newW / 2,
            y = s.height / 2 - newH / 2;

          s.image(userImg, x, y, newW, newH);

          let alpha = 255 * (1 - this.tint);
          s.tint(255, alpha);
          s.image(originalUserImg, x, y, newW, newH);

          s.noTint();
        }
      };

      s.handleUpload = (file) => {
        if (file.type === "image") {
          userImg = s.loadImage(file.data, () => {
            originalUserImg = userImg.get();
            console.log(originalUserImg);
            userImg.filter(s.GRAY);
            s.updateCanvas();
          });
        }
      };

      s.removeImage = () => {
        if (userImg) {
          userImg = null;
          originalUserImg = null;
          s.updateCanvas();
        }
      };

      s.drawPattern = () => {
        for (const [key, value] of Object.entries(this.patterns)) {
          if (value.checked) {
            // first time for a given edition
            if (!(this.edition in loadedPatterns)) {
              loadedPatterns[this.edition] = {};
            }

            // first time for a given social media format
            if (!(this.format in loadedPatterns[this.edition])) {
              loadedPatterns[this.edition][this.format] = {};
            }

            // has this particular pattern been loaded?
            if (!(key in loadedPatterns[this.edition][this.format])) {
              let path = `./img/patterns/${this.edition}/${this.format}/${key}.png`;
              loadedPatterns[this.edition][this.format][key] = s.loadImage(
                path,
                () => {
                  console.log(
                    `Loaded the pattern for ${this.edition} / ${this.format} / ${key}`
                  );
                  s.updateCanvas();
                }
              );
            } else {
              if (this.configRef.editions[this.edition].tintPattern) {
                if (userImg) {
                  s.tint("#ff008f");
                } else {
                  s.tint("#cb0072");
                }
              }
              s.image(
                loadedPatterns[this.edition][this.format][key],
                0,
                0,
                s.width,
                s.height
              );
              s.noTint();
            }
          }
        }
      };

      s.drawLogo = () => {};
      s.drawText = () => {};

      s.draw = () => {
        s.background(0);

        s.drawImages();
        s.drawPattern();

        // s.drawLogo();
        // s.drawText();

        s.noLoop();
      };

      s.updateCanvas = () => {
        s.draw();
      };
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

canvas {
  box-shadow: 0 2px 6px black;
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
  position: fixed;
  left: $p;
  top: 0;
  bottom: 0;
  right: 0;
  background: #666;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>

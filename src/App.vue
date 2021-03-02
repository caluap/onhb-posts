<template>
  <div id="app">
    <div id="desktop" @click="openPanel()">
      <div id="canvas-container"></div>
    </div>
    <div id="toolbar">
      <panel-toggle
        icon-src="./img/icons/format-picker.svg"
        :selected="currentPanel == 'format-picker'"
        @click.native="openPanel('format-picker')"
      />
      <panel-toggle
        icon-src="./img/icons/text.svg"
        :selected="currentPanel == 'text'"
        @click.native="openPanel('text')"
      />
      <panel-toggle
        icon-src="./img/icons/image.svg"
        :selected="currentPanel == 'image'"
        @click.native="openPanel('image')"
      />
      <panel-toggle
        icon-src="./img/icons/pattern-picker.svg"
        :selected="currentPanel == 'pattern-picker'"
        @click.native="openPanel('pattern-picker')"
      />
    </div>
    <panel
      v-show="currentPanel == 'format-picker'"
      :class="{ 'visible-panel': currentPanel == 'format-picker' }"
      title="Seleção de formato"
    >
      <nice-radio
        img-src="img/instagram-brands.svg"
        img-alt="Small camera icon (Instagram)"
        radio-name="social_media_format"
        radio-value="instagram_feed"
        label="Feed"
        @change="changeFormat"
      />
      <nice-radio
        img-src="img/instagram-brands.svg"
        img-alt="Small camera icon (Instagram)"
        radio-name="social_media_format"
        radio-value="instagram_stories"
        label="Stories"
        @change="changeFormat"
      />
      <nice-radio
        img-src="img/facebook-brands.svg"
        img-alt="The banner of an evil empire (Facebook)"
        radio-name="social_media_format"
        radio-value="facebook_feed"
        label="Facebook"
        @change="changeFormat"
      />
      <nice-radio
        img-src="img/twitter-brands.svg"
        img-alt="A small bird (Twitter)"
        radio-name="social_media_format"
        radio-value="twitter_feed"
        label="Twitter"
        @change="changeFormat"
      />
    </panel>
    <panel
      v-show="currentPanel == 'text'"
      :class="{ 'visible-panel': currentPanel == 'text' }"
      title="Caixas de texto"
    >
      <div class="text-alignment-picker">
        <h3 class="label-panel">Alinhamento do texto</h3>
        <input
          type="radio"
          name="text-alignment"
          id="left-alignment"
          v-model="textAlignment"
          value="LEFT"
          checked
        />
        <label for="left-alignment"
          ><img
            src="img/icons/align-left.svg"
            alt="Alinhamento de texto à esquerda."
        /></label>
        <input
          type="radio"
          name="text-alignment"
          id="center-alignment"
          v-model="textAlignment"
          value="CENTER"
        />
        <label for="center-alignment"
          ><img
            src="img/icons/align-center.svg"
            alt="Alinhamento centralizado de texto."
        /></label>
        <input
          type="radio"
          name="text-alignment"
          id="right-alignment"
          v-model="textAlignment"
          value="RIGHT"
        />
        <label for="right-alignment"
          ><img
            src="img/icons/align-right.svg"
            alt="Alinhamento de texto à direita."
        /></label>
      </div>

      <label class="label-panel" for="main-text">Texto principal</label>
      <textarea v-model="mainText" id="main-text" @keyup="update" />
      <label class="label-panel" for="auxiliary-text">Texto auxiliar</label>
      <textarea v-model="auxiliaryText" id="auxiliary-text" @keyup="update" />
    </panel>
    <panel
      v-show="currentPanel == 'image'"
      :class="{ 'visible-panel': currentPanel == 'image' }"
      title="Upload de imagem"
    >
      <div id="img-input-container"></div>
    </panel>
    <panel
      v-show="currentPanel == 'pattern-picker'"
      :class="{ 'visible-panel': currentPanel == 'pattern-picker' }"
      title="Escolha de padrão gráfico tematizado"
      ><template v-for="(pattern, key) in patterns">
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
    </panel>
  </div>
</template>

<script>
import P5 from "p5";
import json from "@/assets/config.json";
import Panel from "./components/Panel";
import PanelToggle from "./components/PanelToggle.vue";
import NiceRadio from "./components/NiceRadio.vue";

export default {
  name: "App",
  data() {
    return {
      currentPanel: "",
      sketch: null,
      p5instance: null,
      textAlignment: "LEFT",
      canvas: null,
      configRef: json,
      tint: 1.0,
      mainText: "",
      auxiliaryText: "",
      dirty: false,
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
    textAlignment: function () {
      this.p5instance.redraw();
    },
  },
  components: { Panel, PanelToggle, NiceRadio },
  updated: function () {
    if (this.dirty) {
      this.dirty = false;
      this.p5instance.updateZoom();
    }
  },
  methods: {
    clickety: function () {
      this.p5instance.redraw();
      this.p5instance.saveImg();
    },
    update: function () {
      this.p5instance.redraw();
    },
    changeFormat: function (newFormat) {
      this.format = newFormat;
    },
    openPanel: function (whichPanel = "") {
      // will close a currently opened panel
      if (this.currentPanel == whichPanel || whichPanel === "") {
        this.currentPanel = "";
        this.dirty = true;
      } else {
        // is opening a currently closed panel space
        if (this.currentPanel == "") {
          this.dirty = true;
        }
        this.currentPanel = whichPanel;
      }
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
        loadedPatterns = {},
        margin = 40;

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
        console.log(`${mainFont}/${auxFont}/${auxFont2}`);
      };

      s.updateZoom = () => {
        // debugger; // eslint-disable-line no-debugger
        let availableHSpace = document.getElementById("desktop").clientHeight,
          availableWSpace = document.getElementById("desktop").clientWidth;

        let el = document.getElementsByClassName("visible-panel");
        if (el.length > 0) {
          let w = el[0].clientWidth;
          availableWSpace -= w;
          document.getElementById(
            "canvas-container"
          ).style.paddingLeft = `${w}px`;
        } else {
          document.getElementById("canvas-container").style.paddingLeft = `0`;
        }

        let ratioH = availableHSpace / this.configRef.formats[this.format].h,
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

      s.drawLogo = () => {
        let logoSize = Math.max(s.width * 0.1, 90);
        let x = s.width - margin - logoSize;
        let y = s.height - margin - logoSize;
        s.image(logo, x, y, logoSize, logoSize);
      };

      s.drawText = () => {
        s.textFont(mainFont);

        let w = (s.width * 2) / 3 - margin;
        let yMainText = 60; //sliderYMainText.value();
        let fsMainText = 40; //sliderFsMainText.value();

        let alignment, x;
        if (this.textAlignment == "LEFT") {
          alignment = s.LEFT;
          x = margin;
        } else if (this.textAlignment == "CENTER") {
          alignment = s.CENTER;
          x = margin;
          w = s.width - 2 * margin;
        } else {
          alignment = s.RIGHT;
          x = s.width - w - margin;
        }
        s.textAlign(alignment, s.TOP);
        s.fill(255);

        s.textSize(fsMainText);
        s.textLeading((fsMainText * 4) / 3);
        s.text(this.mainText, x, yMainText, w, s.height - yMainText);

        let yAuxText = 200;
        let fsAuxText = 20;
        s.textFont(auxFont);
        s.textSize(fsAuxText);
        s.textLeading((fsAuxText * 4) / 3);
        s.text(this.auxiliaryText, x, yAuxText, w, s.height - yAuxText);
      };

      s.draw = () => {
        s.background(0);

        s.drawImages();
        s.drawPattern();

        s.drawLogo();
        s.drawText();

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
@import url("https://fonts.googleapis.com/css2?family=Inter:wght@300..800&display=swap");
@import "@/assets/css/_definitions.scss";

body {
  margin: 0;
  &,
  * {
    font-family: "Inter", sans-serif !important;
    @include fs(0);
  }
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

  z-index: 2;

  padding: 0.5rem;

  background: #333;

  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  @include toolbar_shadow;
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

.radio-container {
  label {
    background-color: rgba(255, 255, 255, 0.5);
    transition: all 0.5s ease;
    padding: 1rem;

    display: flex;
    align-items: center;
    gap: 1rem;

    border-radius: 0.25rem;
    cursor: pointer;
    &:hover {
      background-color: rgba(255, 255, 255, 0.75);
    }
  }

  input {
    display: none;
    &:checked ~ label {
      background-color: white;
    }
  }
  img {
    height: 1.5rem;
  }
}

.text-alignment-picker {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-gap: 0.5rem;
  h3 {
    grid-column: 1 / span 3;
  }
  input {
    display: none;
  }
  label {
    padding: 0.5rem 0.25rem;
    background-color: rgba(255, 255, 255, 0.5);
    transition: all 0.5s ease;
    border-radius: 0.25rem;
    text-align: center;
    cursor: pointer;
    &:hover {
      background-color: rgba(255, 255, 255, 0.75);
    }
  }
  input:checked + label {
    background-color: white;
  }
  img {
    height: 1.5rem;
  }
}

.label-panel {
  font-weight: bold;
}
* + .label-panel {
  margin-top: 1rem;
}

textarea {
  border: none;
  min-height: 8rem;
}
</style>

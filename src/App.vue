<template>
  <div id="app">
    <div id="canvas"></div>
  </div>
</template>

<script>
import P5 from "p5";

export default {
  name: "App",
  components: {},
  created() {
    const sketch = (s) => {
      let w = window.innerWidth;
      let h = window.innerHeight;

      s.setup = () => {
        s.createCanvas(w, h);
      };

      s.draw = () => {
        s.background(33, 33, 33);
        s.ellipse(50, 50, 100, 100);
        s.noLoop();

        // call your method:
        // s.yourMethod(x, y);
      };

      // create methods:
      // s.yourMethod = (x, y) => {
      //   // your method
      // };
    };

    new P5(sketch, "canvas");
  },
};
</script>

<style lang="scss">
body {
  margin: 0;
}
</style>

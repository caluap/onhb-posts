<template>
  <section>
    <div class="title">
      <h2 v-if="title != ''">{{ title }}</h2>
    </div>
    <div class="slot">
      <slot></slot>
    </div>
  </section>
</template>

<script>
export default {
  name: "Panel",
  props: {
    title: {
      type: String,
      default: "",
    },
  },
  data() {
    return {};
  },
};
</script>

<style lang="scss" scoped>
@import "@/assets/css/_definitions.scss";

section {
  position: fixed;
  left: $p;
  top: 0;
  bottom: 0;
  width: $p * 5;

  z-index: 1;

  @include toolbar_shadow;
  background: $panel_c;

  padding: 1rem;
  padding-right: $p/2 + 1rem;
}

.slot {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.title {
  background: $selection;

  position: absolute;
  right: 0;
  bottom: 0;
  top: 0;
  padding: 1rem;
  width: $p/2;

  display: flex;
  flex-direction: column;
  align-items: center;
}
h2 {
  color: white;
  writing-mode: vertical-lr;

  font-weight: 500;
  @include fs(0);
}
</style>
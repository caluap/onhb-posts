<template>
  <section>
    <h2 v-if="title != ''">{{ title }}</h2>
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
}

.slot {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

h2 {
  display: block;
  background: $selection;
  color: white;
  writing-mode: vertical-lr;

  position: absolute;
  right: 0;
  bottom: 0;
  top: 0;
  padding: 1rem 0.5rem;

  font-weight: 600;
  @include fs(1);
}
</style>
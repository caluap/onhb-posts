<template>
  <button :class="{ selected }">
    <img :src="iconSrc" :alt="altText" />
  </button>
</template>

<script>
export default {
  name: "PanelToggle",
  methods: {},
  props: {
    iconSrc: { type: String },
    altText: { type: String, default: "" },
    selected: { type: Boolean, default: false },
  },
};
</script>

<style lang="scss" scoped>
@import "@/assets/css/_definitions.scss";

button {
  padding: 0.5rem;
  border: none;
  background-color: white;
  border-radius: 0.25rem;
  aspect-ratio: 1 / 1;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  user-select: none;
  &:hover {
    background: coral;
  }
  &.selected {
    background-color: $selection;
  }
}
img {
  width: 1.5rem;
}
</style>
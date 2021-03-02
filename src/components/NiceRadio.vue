<template>
  <div>
    <input type="radio" :id="id" @change="update" :name="radioName" />
    <label :for="id" :class="{ checked }"
      ><img v-if="imgSrc != ''" :src="imgSrc" :alt="imgAlt" />
      {{ label }}</label
    >
  </div>
</template>

<script>
export default {
  name: "NiceRadio",
  props: {
    imgSrc: { type: String, default: "" },
    imgAlt: {},
    radioName: {},
    radioValue: {},
    model: {},
    label: {},
    checked: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      id: null,
    };
  },
  methods: {
    update: function () {
      this.$emit("change", this.radioValue);
    },
  },
  computed: {},
  mounted() {
    this.id = this._uid;
  },
};
</script>

<style lang="scss" scoped>
div {
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
    &.checked {
      background-color: white;
    }
  }

  input {
    display: none;
  }
  img {
    height: 1.5rem;
  }
}
</style>
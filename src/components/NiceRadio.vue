<template>
  <div>
    <input
      type="radio"
      :id="id"
      v-model="radioButtonValue"
      :value="radioValue"
      :name="radioName"
    />
    <label :for="id"><img :src="imgSrc" :alt="imgAlt" /> {{ label }}</label>
  </div>
</template>

<script>
export default {
  name: "NiceRadio",
  props: ["imgSrc", "imgAlt", "radioName", "radioValue", "model", "label"],
  data() {
    return {
      id: null,
    };
  },
  computed: {
    radioButtonValue: {
      get: function () {
        return this.radioValue;
      },
      set: function () {
        // Communicate the change to parent component so that selectedValue can be updated
        this.$emit("change", this.radioValue);
      },
    },
  },
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
</style>
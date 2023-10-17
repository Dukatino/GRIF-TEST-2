<template>
  <div>
    <label v-for="(option, index) in options" :key="index">
      <input
        type="radio"
        :id="option"
        :value="option"
        v-model="selectedOption"
      />
      {{ option }}
    </label>
  </div>
</template>

<script>
export default {
  props: {
    options: {
      type: Array,
      required: true,
    },
    selected: {
      type: String,
      default: "win",
    },
  },
  data() {
    return {
      selectedOption: this.selected,
    };
  },
  watch: {
    selected(newVal) {
      this.selectedOption = newVal;
    },
    selectedOption(newOption) {
      this.$emit("selected", newOption);
    },
  },
};
</script>

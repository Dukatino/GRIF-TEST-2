<template>
  <div>
    <table class="spacing-top">
      <tr>
        <th>#</th>
        <th>match</th>
        <th>1</th>
        <th>x</th>
        <th>2</th>
        <th>bet</th>
        <th>bank</th>
      </tr>

      <tr
        v-for="(match, index) in teamData"
        :key="index + teamName + '-' + match.match"
      >
        <td style="font-size: 10px; color: gray; text-align: center">
          {{ teamName.slice(0, 3) }}
          <span class="d-block">{{ index + 1 }}</span>
        </td>
        <td>{{ match.match }}</td>
        <td
          :class="{
            win: match['1-bool'],
            'bet-event':
              (betOn.win && match.home) ||
              (betOn.loss && !match.home) ||
              isOddInRange(match['1'], oddRange),
          }"
        >
          {{ match["1"] }}
        </td>
        <td
          :class="{
            win: match['x-bool'],
            'bet-event': betOn.draw || isOddInRange(match['x'], oddRange),
          }"
        >
          {{ match["x"] }}
        </td>
        <td
          :class="{
            win: match['2-bool'],
            'bet-event':
              (betOn.win && !match.home) ||
              (betOn.loss && match.home) ||
              isOddInRange(match['2'], oddRange),
          }"
        >
          {{ match["2"] }}
        </td>
        <td>{{ match.bet }}</td>
        <td>{{ match.bank }}</td>
      </tr>
    </table>
  </div>
</template>

<script>
import isEmptyObject from "../utility/isEmptyObject";
export default {
  name: "Table",
  props: {
    teamData: {
      type: Array,
      default: () => [],
    },
    teamName: {
      type: String,
      required: true,
    },
    betOn: {
      type: Object,
      default: () => {},
    },
    oddRange: {
      type: Object,
      default: () => {},
    },
    // isOddInRange: {
    //   type: Function,
    //   default: () => {},
    // },
  },
  methods: {
    isOddInRange(odd, oddRange) {
      if (!this.betOn.range) {
        return false;
      }
      if (!isEmptyObject(oddRange)) {
        return oddRange.min <= odd && odd <= oddRange.max;
      }
      return false;
    },
  },
  // data() {
  //   return {
  //     betInput: this.bet,
  //   };
  // },
  // watch: {
  //   betInput() {
  //     this.$emit('bet-input', this.betInput);
  //   },
  // },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
.win {
  background: lightyellow;
}
.bet-event {
  border: 1px dashed lighten(gray, 10%);
}
.win.bet-event {
  background: lawngreen;
}
tr:nth-child(odd) {
  background-color: rgba(black, 0.06);
}
td {
  padding: 4px;
}
</style>

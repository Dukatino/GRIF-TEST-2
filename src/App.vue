<template>
  <div id="app">
    <button
      type="button"
      class="filter-button spacing-right d-flex items-center"
      @click="open"
    >
      <img
        src="./assets/graphics/icons/icon-cog.svg"
        alt=""
        style="width: 16px"
      />
      &nbsp;Settings
    </button>
    <BottomSheet ref="bottomSheet">
      <div class="spacing" style="margin-bottom: 100px">
        <div>
          <span>TOTAL:&nbsp;</span>
          <b class="inline-block ml-4">{{ srajtofla }} </b>
          <div class="d-inline-flex lh-1" style="flex-wrap: wrap">
            <span class="text-xxs spacing-left-sm"
              >adj.: {{ oddsPercentageAdjustment * 100 }}%</span
            >
            <span
              v-if="
                betOn.range &&
                oddsRangeFilter &&
                oddsRangeFilter.min &&
                oddsRangeFilter.max
              "
              class="d-inline-flex text-xxs spacing-left-sm"
              >&nbsp;|&nbsp;
              <span
                >odds:{{ oddsRangeFilter.min }}-{{ oddsRangeFilter.max }}</span
              >
              &nbsp;|&nbsp;
              <span>range filtered events: {{ oddsRangeCount }}</span>
            </span>

            <span
              v-if="!betOn.range && insights && insights.convertedodds"
              class="text-xxs spacing-left-sm"
            >
              <span>| converted: {{ insights.convertedodds.length }}</span>
            </span>
            <span
              v-if="oddsInsights && oddsInsights.hasOwnProperty('count')"
              class="text-xxs spacing-left-sm"
              >| all events: {{ allEventsCount }}</span
            >
            <span
              v-if="oddsInsights && oddsInsights.hasOwnProperty('mean')"
              class="text-xxs spacing-left-sm"
            >
              | mean av.: {{ oddsInsights.mean }}</span
            >
            <span
              v-if="oddsInsights && oddsInsights.hasOwnProperty('median')"
              class="text-xxs spacing-left-sm"
            >
              | median av.: {{ oddsInsights.median }}</span
            >
          </div>
        </div>
        <h4 class="spacing-top">Bet on team's:</h4>
        <RadioOptions
          class="spacing-top"
          @selected="setBetOn"
          :selected="selectedBetOn"
          :options="['win', 'draw', 'loss', 'range']"
        />
        <RangeOddsCount
          v-if="betOn.range"
          class="spacing-top"
          :range="oddsRangeFilter"
          @range="setOddsRangeFilter"
          :odds-range-count="oddsRangeCount"
          :odds-count="oddsInsightsCount"
        />
        <div class="spacing-top">
          <h4>Bet:</h4>
          <input
            class="spacing-top"
            style="max-width: 50px"
            type="number"
            v-model="bet"
          /><span> €</span>
        </div>

        <TeamSelector
          class="spacing-top"
          :selected-teams="selectedTeams"
          :teams="listOfTeams"
          @update:selectedTeams="setSelectedTeams"
        />
      </div>
    </BottomSheet>
    <div class="bank" v-cloak>
      <div class="d-flex flex-column">
        <div class="d-flex justify-center">
          <span>
            <span>TOTAL:&nbsp;</span>
            <b class="inline-block ml-4">{{ srajtofla }} </b>
          </span>
          <span
            >&nbsp;|&nbsp;
            <template v-if="betOn.win">BET ON WIN</template>
            <template v-else-if="betOn.loss">BET ON LOSS</template>
            <template v-else-if="betOn.draw">BET ON DRAW</template>
            <template v-else>BET ON RANGE</template>
          </span>
        </div>
        <div class="d-flex lh-1 justify-center">
          <span class="text-xxs spacing-left-sm"
            >adj.: {{ oddsPercentageAdjustment * 100 }}%</span
          >
          <span
            v-if="
              betOn.range &&
              oddsRangeFilter &&
              oddsRangeFilter.min &&
              oddsRangeFilter.max
            "
            class="d-inline-flex text-xxs spacing-left-sm"
            >&nbsp;|&nbsp;
            <span
              >odds:{{ oddsRangeFilter.min }}-{{ oddsRangeFilter.max }}</span
            >
            &nbsp;|&nbsp;
            <span>range filtered events: {{ oddsRangeCount }}</span>
          </span>
          <span
            v-if="!betOn.range && insights && insights.convertedodds"
            class="text-xxs spacing-left-sm"
          >
            <span>| converted: {{ insights.convertedodds.length }}</span>
          </span>
          <span
            v-if="oddsInsights && oddsInsights.hasOwnProperty('count')"
            class="text-xxs spacing-left-sm"
            >| all events: {{ allEventsCount }}</span
          >
        </div>
        <div class="d-flex lh-1 justify-center">
          <span
            v-if="oddsInsights && oddsInsights.hasOwnProperty('mean')"
            class="text-xxs spacing-left-sm"
            >mean av.: {{ oddsInsights.mean }}</span
          >
          <span
            v-if="oddsInsights && oddsInsights.hasOwnProperty('median')"
            class="text-xxs spacing-left-sm"
          >
            | median av.: {{ oddsInsights.median }}</span
          >
        </div>
      </div>
    </div>

    <div class="tables">
      <div v-for="(teamData, teamName) in teamsDataFiltered" :key="teamName">
        <Table
          :team-data="teamData"
          :team-name="teamName"
          :bet-on="betOn"
          :odd-range="oddsRangeFilter"
          :is-odd-in-range="isOddInRange"
        />
      </div>
    </div>
  </div>
</template>

<script>
import Table from "./components/Table";
import RangeOddsCount from "./components/RangeOddsCount";
import RadioOptions from "./components/RadioOptions";
import TeamSelector from "./components/TeamSelector";
import BottomSheet from "@nclsm/vue-bottom-sheet-vue2";
import teamsData from "./assets/data/teams-2022-2023-data.json";
import { listOfTeams } from "./config";
import odds from "./assets/mixins/odds";
// import cookies from "./assets/mixins/cookies";
import findTrueObjectProperty from "./utility/findTrueObjectProperty";
import isEmptyObject from "./utility/isEmptyObject";

export default {
  name: "App",
  components: {
    Table,
    RangeOddsCount,
    RadioOptions,
    BottomSheet,
    TeamSelector,
  },
  // mixins: [odds, cookies],
  mixins: [odds],
  data: function () {
    return {
      initMoney: 1,
      oddsPercentageAdjustment: 0.95,
      moneyPerMatch: 3,
      bank: 0,
      bet: this.initMoney,
      srajtofla: 0,
      insights: {
        convertedodds: [],
      },
      betOn: {
        win: true,
        loss: false,
        draw: false,
        range: false,
      },
      allEventsCount: 0,
      selectedBetOn: "",
      selectedTeams: listOfTeams,
    };
  },
  computed: {
    listOfTeams: () => listOfTeams,
    teamsDataMapped() {
      let newData = {};
      this.allEventsCount = 0;
      this.bank = 0;
      this.insights.convertedodds = [];

      for (let i = 0; i < this.selectedTeams.length; i++) {
        this.allEventsCount += teamsData[this.selectedTeams[i]].length;

        newData[this.selectedTeams[i]] = teamsData[this.selectedTeams[i]].map(
          (match) => {
            const adjustedOdd1 = this.getAdjustedOdd(match["1"]);
            const adjustedOddX = this.getAdjustedOdd(match["x"]);
            const adjustedOdd2 = this.getAdjustedOdd(match["2"]);
            let bet = this.bet;

            if (!this.insights.convertedodds) {
              this.insights.convertedodds = [];
            }

            if (this.betOn.win) {
              if (match.home) {
                if (!match["1-bool"]) {
                  this.bank -= this.bet;
                } else {
                  this.bank += this.bet * adjustedOdd1 - this.bet;
                  this.insights.convertedodds.push(adjustedOdd1);
                }
              } else {
                if (!match["2-bool"]) {
                  this.bank -= this.bet;
                } else {
                  this.bank += this.bet * adjustedOdd2 - this.bet;
                  this.insights.convertedodds.push(adjustedOdd2);
                }
              }
            } else if (this.betOn.loss) {
              if (match.home) {
                if (!match["2-bool"]) {
                  this.bank -= this.bet;
                } else {
                  this.bank += this.bet * adjustedOdd2 - this.bet;
                  this.insights.convertedodds.push(adjustedOdd2);
                }
              } else {
                if (!match["1-bool"]) {
                  this.bank -= this.bet;
                } else {
                  this.bank += this.bet * adjustedOdd1 - this.bet;
                  this.insights.convertedodds.push(adjustedOdd1);
                }
              }
            } else if (this.betOn.draw) {
              if (!match["x-bool"]) {
                this.bank -= this.bet;
              } else {
                this.bank += this.bet * adjustedOddX - this.bet;
                this.insights.convertedodds.push(adjustedOddX);
              }
            } else if (this.betOn.range) {
              let winningOdd;
              bet = 0;

              if (match["x-bool"]) {
                winningOdd = adjustedOddX;
              } else if (match["1-bool"]) {
                winningOdd = adjustedOdd1;
              } else if (match["2-bool"]) {
                winningOdd = adjustedOdd2;
              }

              [adjustedOdd1, adjustedOddX, adjustedOdd2].forEach(
                (odd, index) => {
                  if (this.isOddInRange(odd, this.oddsRangeFilter)) {
                    this.bank -= this.bet;
                    bet += this.bet;
                  }
                }
              );
              if (
                ![adjustedOdd1, adjustedOddX, adjustedOdd2].some((odd) =>
                  this.isOddInRange(odd, this.oddsRangeFilter)
                )
              ) {
                bet = 0;
              }

              if (this.isOddInRange(winningOdd, this.oddsRangeFilter)) {
                this.bank += this.bet * winningOdd;
                this.insights.convertedodds.push(winningOdd);
              }
            }

            return {
              ...match,
              1: adjustedOdd1,
              2: adjustedOdd2,
              x: adjustedOddX,
              bet: bet,
              bank: Math.round(this.bank * 100) / 100,
            };
          }
        );
      }
      this.srajtofla = Math.round(this.bank * 100) / 100 + " EUR";
      this.insights.convertedodds.sort();
      return newData;
    },
    teamsDataFiltered() {
      // Create a new object to store filtered data
      const filteredData = {};

      for (const [teamName, teamData] of Object.entries(this.teamsDataMapped)) {
        if (
          this.selectedTeams.length === 0 ||
          this.selectedTeams.includes(teamName)
        ) {
          // Filter and add data for the selected team
          const filteredTeamData = teamData.filter((item) => {
            // if (this.selectedProperties.length === 0) return true;
            // if (this.selectedProperties.includes('1')) {
            //   const val1 = parseFloat(item['1']);
            //   if (val1 < this.minValue || val1 > 1.7) return false;
            // }
            // if (this.selectedProperties.includes('2')) {
            //   const val2 = parseFloat(item['2']);
            //   if (val2 < this.minValue || val2 > 1.7) return false;
            // }
            return true;
          });

          if (filteredTeamData.length > 0) {
            filteredData[teamName] = filteredTeamData;
          }
        }
      }

      return filteredData;
    },
  },
  created() {
    this.bet = this.initMoney;
  },
  mounted() {
    this.selectedBetOn = findTrueObjectProperty(this.betOn);
  },
  methods: {
    open() {
      this.$refs.bottomSheet.open();
    },
    close() {
      this.$refs.bottomSheet.close();
    },
    computeBet(odd, debt) {
      return debt / (odd - 1);
    },
    setBetOn(type) {
      this.betOn = {
        win: false,
        draw: false,
        loss: false,
        range: false,
      };
      if (type === "win") {
        this.betOn.win = true;
      } else if (type === "loss") {
        this.betOn.loss = true;
      } else if (type === "draw") {
        this.betOn.draw = true;
      } else if (type === "range") {
        this.betOn.range = true;
      }
      // trigger main computed teamsDataMapped
      this.bank = 0;
    },

    setInsight(key, value) {
      this.insights[key] = value;
    },
    setSelectedTeams(newSelectedTeams) {
      this.selectedTeams = newSelectedTeams;
    },
    getAdjustedOdd(odd) {
      return Math.round(odd * this.oddsPercentageAdjustment * 100) / 100;
    },
  },
};
</script>

<style>
body {
  background-color: rgba(128, 128, 128, 0.06);
  font-family: "Franklin Gothic Medium", "Arial Narrow", Arial, sans-serif;
}
body,
p,
h1,
h2,
h3,
h4,
h5 {
  margin: 0;
}
v-cloak {
  display: none;
}

#app {
  display: flex;
  justify-content: center;
}

/* .teams-wrapper {
  flex-direction: column;
  align-items: flex-start;
} */
.tables {
  padding-top: 40px;
  font-size: 14px;
}
.bank {
  position: fixed;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  top: 0;
  padding: 5px;
  width: 100%;
  background: white;
  text-align: left;
  box-shadow: 0 3px 8px rgba(128, 128, 128, 0.11);
}
.spacing {
  margin: 12px;
}
.spacing-top {
  margin-top: 12px;
}
.spacing-left {
  margin-left: 12px;
}
.spacing-right {
  margin-right: 12px;
}
.spacing-left-sm {
  margin-left: 4px;
}
.spacing-right-sm {
  margin-right: 4px;
}
.filter-button {
  position: fixed;
  bottom: 20px;
  left: 16px;
  background-color: white;
  color: black;
  border: 1px solid gray;
  padding: 10px 15px;
  cursor: pointer;
  font-size: 16px;
  border-radius: 4px;
}
.d-block {
  display: block;
}
.d-inline-block {
  display: inline-block;
}
.d-flex {
  display: flex;
}
.d-inline-flex {
  display: inline-flex;
}
.flex-column {
  flex-direction: column;
}
.items-center {
  align-items: center;
}
.justify-center {
  justify-content: center;
}
.text-xxs {
  font-size: 10px;
}
.text-xs {
  font-size: 12px;
}
.text-small {
  font-size: 14px;
}
.text-normal {
  font-size: 16px;
}
.text-xl {
  font-size: 20px;
}
.lh-1 {
  line-height: 1;
}
.lh-small {
  line-height: 1.2;
}
</style>

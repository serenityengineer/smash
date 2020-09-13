<template>
  <div>
    <Header v-if="online"
      title="Pods"
      prev="draft"
      next="2v2"
      :disabled="!bloodbath_locked"
      :prevDisabled="lockedPods > 0 && lockedPods < 8"
    />
    <Header v-else
      title="Solo Cup Seeding"
      prev="2v2"
      next="1v1"
      :disabled="!one_vs_one_seeding_locked"
      :prevDisabled="lockedPods > 0 && lockedPods < 8"
    />
    <div class="row">
      <div class="col-3">
        <table class="table table-sm">
          <thead>
            <tr>
              <th scope="col">#</th>
              <th scope="col">Player</th>
              <th scope="col">Score</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(player, index) in sortedPlayers" :key="player.id" @click="incrementPodScore(player)">
              <th scope="row">{{index + 1}}</th>
              <td scope="row"><PlayerCard :player="player"/></td>
              <th scope="row" v-bind:class="{tied: needsTieBreaker(player)}">{{player.results.podScore.toFixed(1)}}</th>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="col">
        <div class="row my-3">
          <div class="col"><Pod title="Doms" :pool="doms" @locked="lockedPods += 1" @unlocked="lockedPods--"/></div>
          <div class="col"><Pod title="Subs" :pool="subs" @locked="lockedPods += 1" @unlocked="lockedPods--"/></div>
          <div class="col"><Pod title="Rods" :pool="rods" @locked="lockedPods += 1" @unlocked="lockedPods--"/></div>
          <div class="col"><Pod title="Tacos" :pool="tacos" @locked="lockedPods += 1" @unlocked="lockedPods--"/></div>
          <div class="col"><Pod title="Waifus" :pool="waifus" @locked="lockedPods += 1" @unlocked="lockedPods--"/></div>
        </div>
        <div class="row">
          <div class="col"><Pod title="Husbandos"
            :pool="husbandos" @locked="lockedPods += 1" @unlocked="lockedPods--"/></div>
          <div class="col"><Pod title="Zeke's Team"
            :pool="zekesteam" @locked="lockedPods += 1" @unlocked="lockedPods--"/></div>
          <div class="col"><Pod title="Russ's Team"
            :pool="russsteam" @locked="lockedPods += 1" @unlocked="lockedPods--"/></div>
          <div class="col"><Pod title="Zoomers"
            :pool="zoomers" @locked="lockedPods += 1" @unlocked="lockedPods--"/></div>
          <div class="col"><Pod title="Boomers"
            :pool="boomers" @locked="lockedPods += 1" @unlocked="lockedPods--"/></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {
  mapState, mapGetters, mapActions, mapMutations,
} from 'vuex';
import Header from '../Header.vue';
import PlayerCard from '../PlayerCard.vue';
import Pod from '../1v1/Pod.vue';

export default {
  name: 'Pods',
  components: {
    Pod,
    PlayerCard,
    Header,
  },
  data() {
    return {
      lockedPods: 0,
      gameOver: false,
    };
  },
  computed: {
    ...mapState(['bloodbath', 'online']),
    ...mapGetters(['sortedPlayerList']),
    ...mapGetters('results', ['one_vs_one_seeding_locked', 'bloodbath_locked']),
    players() {
      return this.online ? [...this.sortedPlayerList] : [...this.bloodbath];
    },
    sortedPlayers() {
      return [...this.players].sort((a, b) => {
        if (a.results.podScore === 0) return 1;
        if (b.results.podScore === 0) return -1;

        return a.results.podScore < b.results.podScore ? -1 : 1;
      });
    },
    doms() {
      return this.players.slice(0, 4);
    },
    subs() {
      return this.players.slice(4);
    },
    rods() {
      return this.players.filter((player, index) => index > 1 && index < 6);
    },
    waifus() {
      return this.players.filter((player, index) => index < 2 || index > 5);
    },
    husbandos() {
      return this.players.filter((player, index) => index === 0 || index === 1 || index === 4 || index === 5);
    },
    tacos() {
      return this.players.filter((player, index) => index === 2 || index === 3 || index === 6 || index === 7);
    },
    zekesteam() {
      return this.players.filter((player, index) => index === 1 || index === 3 || index === 5 || index === 6);
    },
    russsteam() {
      return this.players.filter((player, index) => index === 0 || index === 3 || index === 4 || index === 7);
    },
    zoomers() {
      return this.players.filter((player, index) => index === 1 || index === 2 || index === 4 || index === 7);
    },
    boomers() {
      return this.players.filter((player, index) => index === 0 || index === 2 || index === 5 || index === 6);
    },
  },
  watch: {
    lockedPods(newValue, oldValue) {
      if (newValue === 10) {
        if (!this.hasDuplicatePodScores()) {
          this.gameOver = true;
        }
      } else if (oldValue === 10 && this.gameOver) {
        this.gameOver = false;
        this.set1v1SeedingResults([]);
      }
    },
    gameOver() {
      if (this.gameOver) {
        if (this.online) {
          this.setBloodbathResults([...this.sortedPlayers]);
        } else {
          this.set1v1SeedingResults([...this.sortedPlayers]);
          this.update1v1Round1Games();
        }

        this.updatePodScores([...this.sortedPlayers].reverse());
      }
    },
  },
  methods: {
    ...mapMutations(['updatePodScores']),
    ...mapMutations('results', ['set1v1SeedingResults', 'setBloodbathResults']),
    ...mapActions(['updatePlayerPodScore']),
    ...mapActions('results', ['update1v1Round1Games']),
    hasDuplicatePodScores() {
      const podScores = this.sortedPlayers.map((player) => player.results.podScore);
      return podScores.some((item, index) => podScores.indexOf(item) !== index);
    },
    hasDuplicatePodScore(value) {
      const podScores = this.sortedPlayers.map((player) => player.results.podScore);
      const { podScore } = value.results;

      return podScores.indexOf(podScore) !== podScores.lastIndexOf(podScore);
    },
    incrementPodScore(player) {
      if (this.needsTieBreaker(player)) {
        this.updatePlayerPodScore({ id: player.id, score: -0.1 });
        if (!this.hasDuplicatePodScores()) {
          this.gameOver = true;
        }
      }
    },
    needsTieBreaker(player) {
      return this.lockedPods === 10 && this.hasDuplicatePodScore(player);
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.tied {
  background-color: lightcoral;
}
</style>

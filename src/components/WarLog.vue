<template>
  <v-container fluid>
    <v-row class="text-center">
      <v-col cols="12" align="center">
        <v-progress-circular
          :size="50"
          color="secondary"
          indeterminate
          v-if="transformed.length == 0"
        ></v-progress-circular>
        <v-responsive  :max-width="600">
          <v-card
            class="player mb-2"
            v-for="(player, i) in transformed"
            :key="i"
            flat
          >
            <div class="d-flex justify-space-between">
              <div class="d-flex">
                <div
                  class="ranking d-flex align-center ma-2 pa-1 subtitle-2" 
                  :class="i+1 == 1 ? 'gold' : i+1 == 2 ? 'silver' : i+1 == 3 ? 'bronze' : 'plain'"
                >
                  #{{i+1}}
                </div>
                <v-card-title
                  class="subtitle-2 px-0 pa-0 text-no-wrap"
                  v-text="player.name"
                ></v-card-title>
              </div>
              <div class="wars d-flex align-center mr-2">
                <div 
                  v-for="(war, j) in player.wars"
                  :key="j"
                  class="d-flex ml-1 log align-top"
                  v-bind:class="logClass(war)"
                >
                  <v-badge
                    bordered
                    v-if="war.collectionDayBattlesPlayed < 3"
                    color="warning"
                    class="ml-2 mt-1"
                    dot
                  >
                  </v-badge>
                </div>
              </div>
            </div>
          </v-card>
        </v-responsive>
      </v-col>
    </v-row>
    <v-row class="text-center" v-if="transformed.length > 0">
      <v-col cols="12" align="center">
        <v-card
          class="legend mb-1"
          min-width="400"
          max-width="600"
          v-for="(legendItem, i) in legend"
          :key="i"
          flat
        >
          <div class="d-flex justify-space-between">
            <div class="d-flex align-center">
              <div 
                class="d-flex log align-top ma-2 pa-1"
                :class="legendItem.class"
              >
                <v-badge
                  bordered
                  v-if="legendItem.type == 'badge'"
                  color="warning"
                  class="ml-1"
                  dot
                >
                </v-badge>
              </div>
              <span class="caption">{{legendItem.description}}</span>
            </div>
          </div>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
  import axios from 'axios'

  export default {
    name: 'WarLog',
    props: {
      clanTag: String,
    },
    data: () => ({
      error: 'error',
      transformed: [],
      legend: [
        {
          'type': 'log',
          'class': 'all',
          'description': 'Batalles final i de recol·lecció guanyades'
        },
        {
          'type': 'log',
          'class': 'win',
          'description': 'Batalla final guanyada'
        },
        {
          'type': 'log',
          'class': 'lost',
          'description': 'Batalla final perduda'
        },
        {
          'type': 'log',
          'class': 'skipped',
          'description': 'Batalla final no feta'
        },
        {
          'type': 'log',
          'class': '',
          'description': 'No participa a la guerra'
        },
        {
          'type': 'badge',
          'class': 'none',
          'description': 'Alguna batalla de recol·lecció no feta'
        },
      ]
    }),
    methods: {
      logClass: function (warlog) {
        if (warlog.won > 0) {
          if (warlog.maxCollectionPointsReached) {
            return 'all'
          }
          return 'win'
        }
        if (warlog.won == 0) {
          if (warlog.lost > 0) {
            return 'lost'
          } else {
            return 'skipped'
          }
        }
        return null
      },
    },
    created() {
      axios
        .get('https://croyale-api.herokuapp.com/clan/'+this.clanTag+'/warlog')
        .then(response => {
          this.transformed = response.data.warlog
          this.$emit('updateLastWarDate', response.data.lastWarEndDate);
        })
        .catch(error => {
          this.error = error.response
        })
    }
  }
</script>

<style scoped>
.v-card.player {
  background: linear-gradient(to bottom, #F1F1F1 0%,#F1F1F1 50%,#F1F1F1 50%,#E1E1E1 50%,#E1E1E1 100%);
}
.v-card.player .headline {
  color: #333;
  line-height: 0;
}
.v-card.player .ranking {
  background: #444;
  color: #FFF;
  border-radius: 4px
}
.v-card.player .ranking.gold {
  background: #FFB800;
}
.v-card.player .ranking.silver {
  background: #CECECE;
}
.v-card.player .ranking.bronze {
  background: #BC6618;
}
.v-card.legend {
  color: #333;
}
.log{
  width: 12px;
  height: 12px;
  background: #888;
  border-radius: 4px;
}
.log.win{
  background: #14BE44;
}
.log.lost{
  background: #D13333;
}
.log.all{
  background: #00A3FF;
}
.log.skipped{
  background: #424242;
}
.log.none{
  background: transparent;
  border: 1px solid #DDD;
}
</style>

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
        <v-card
          class="player mb-2"
          v-for="(player, i) in transformed"
          :key="i"
          flat
        >
          <div class="d-flex justify-space-between">
            <div class="d-flex">
              <div class="ranking d-flex align-center ma-2 pa-1">
                #{{i+1}}
              </div>
              <v-card-title
                class="headline px-0"
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
              <span>{{legendItem.description}}</span>
            </div>
          </div>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
  //import clan from './../../resources/clan.json'
  //import warlog from './../../resources/warlog.json'
  import axios from 'axios'

  export default {
    name: 'WarLog',

    data: () => ({
      //clan: clan,
      //warlog: warlog,
      transformed: [],
      legend: [
        {
          'type': 'log',
          'class': 'all',
          'description': 'Batalla final i batalles de recol·lecció guanyades'
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
      calculateScore: function(warlogParticipant) {
        var score = 0
        if (warlogParticipant.cardsEarned == 2805 || warlogParticipant.cardsEarned == 2640) {
          score += 500
        }
        if (warlogParticipant.battlesPlayed > 0) {
          if (warlogParticipant.wins == 1) {
            score += 3500
          }
          if (warlogParticipant.wins == 2) {
            score += 3500 + 1500
          }
        } else {
          score -= 2000
        }
        score -= 500 * (3 - warlogParticipant.collectionDayBattlesPlayed)
        return score
      },
      getWarForPlayer: function(warlogParticipant, warCreatedDate) {
        var war = {
          'createdDate': warCreatedDate,
        }
        if (warlogParticipant.cardsEarned == 2805 || warlogParticipant.cardsEarned == 2640) {
          war.maxCollectionPointsReached = true
        } else {
          war.maxCollectionPointsReached = false
        }
        war.won = warlogParticipant.wins
        war.lost = warlogParticipant.battlesPlayed - warlogParticipant.wins
        war.collectionDayBattlesPlayed = warlogParticipant.collectionDayBattlesPlayed
        war.score = this.calculateScore(warlogParticipant)
        return war
      },
      sortPlayersByScore: function(players) {
        var sortable = [];
        for (var tag in players) {
          sortable.push([tag, players[tag].score]);
        }
        sortable.sort(function(a, b) {
          return b[1] - a[1];
        });
        sortable.forEach(sorted => {
          var tag = sorted[0]
          if (Object.keys(players[tag].wars).length > 0) {
            this.transformed.push(players[tag])
          }
        })
      },
      addMissingWars: function(warCreatedDates) {
        this.transformed.forEach(p => {
          warCreatedDates.forEach(date => {
            if (!(date in p.wars)) {
              p.wars[date] = {
                'createdDate': date,
              }
            }
          })
          var sortableWars = []
          for (var warDate in p.wars) {
            sortableWars.push(warDate)
          }
          sortableWars.sort()
          sortableWars.reverse()
          var sortedWars = {}
          sortableWars.forEach(sortedWar => {
            sortedWars[sortedWar] = p.wars[sortedWar]
          })
          p.wars = sortedWars
        })
      },
      transformData: function() {
        var warCreatedDates = []
        var players = {}
        this.clan.items.forEach(p => players[p.tag] = {
          'name': p.name,
          'score': 0,
          'wars': {}
          })
        this.warlog.items.forEach(w => {
          if (!(w.createdDate in warCreatedDates)) {
            warCreatedDates.push(w.createdDate)
          }
          w.participants.forEach(participant => {
            if (participant.tag in players) {
              var war = this.getWarForPlayer(participant, w.createdDate)
              players[participant.tag].wars[w.createdDate] = war
              players[participant.tag].score += war.score
            }
          })
        })
        this.$emit('updateLastWarDate', warCreatedDates[0]);
        this.sortPlayersByScore(players)
        this.addMissingWars(warCreatedDates)
      },
    },
    created() {
      //this.transformData()
      axios
        .get('http://130.211.26.158/clan/2PUGVU8U/warlog')
        .then(response => {
          this.transformed = response.data.warlog
          this.$emit('updateLastWarDate', response.data.lastWarEndDate);
        })
        .catch(error => {
          console.log(error.response)
        })
    }
  }
</script>

<style scoped>
.v-card.player {
  background: linear-gradient(to bottom, #F1F1F1 0%,#F1F1F1 50%,#F1F1F1 50%,#E1E1E1 50%,#E1E1E1 100%);
  /*min-width: 500px;*/
  max-width: 800px;
}
.v-card.player .headline {
  color: #333;
  line-height: 0;
}
.v-card.player .ranking {
  background: #444;
  color: #FFF;
  font-size: 12px;
  border-radius: 4px
}
.v-card.legend {
  font-size: 10px;
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

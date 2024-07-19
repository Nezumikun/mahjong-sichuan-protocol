<template lang="pug">
v-sheet(
  class="d-flex"
  width="300"
  v-if="displaySettings.needInitPlayer"
) 
  v-form(
    ref="initPlayers"
  )
    v-text-field(
      v-model="formInitPlayers.players[0]"
      :label="formInitPlayers.random ? 'Игрок 1' : '東 Восток'"
      required
    )
    v-text-field(
      v-model="formInitPlayers.players[1]"
      :label="formInitPlayers.random ? 'Игрок 2' : '南 Юг'"
      required
    )
    v-text-field(
      v-model="formInitPlayers.players[2]"
      :label="formInitPlayers.random ? 'Игрок 3' : '西 Запад'"
      required
    )
    v-text-field(
      v-model="formInitPlayers.players[3]"
      :label="formInitPlayers.random ? 'Игрок 4' : '北 Север'"
      required
    )
    v-checkbox(
      v-model="formInitPlayers.random"
      label="Случайный выбор места"
      required
    )
    div(
      class="d-flex flex-column"
    )
      v-btn(
        class="mt-4"
        color="success"
        block
        @click="initPlayers"
      ) Начать
      v-btn(
        class="mt-4"
        color="error"
        block
        to="/"
      ) Отмена

div(
  v-if="!displaySettings.needInitPlayer"
)
  div(
    v-if="displaySettings.roundInProggress"
  )
    v-btn(
      class="mt-4"
      color="success"
      block
      @click="displayMahjongDialog"
    ) Маджонг {{ gameState.mahjongCount + 1 }}
    v-btn(
      class="mt-4"
      color="warning"
      block
      @click="displayKongDialog"
    ) Конг
    v-btn(
      class="mt-4"
      color="error"
      block
      @click="endOfWall"
    ) Закончилась стена
  v-data-table(
    :headers="displaySettings.headers"
    :items="gameState.rounds"
    item-key="number"
    items-per-page="-1"
    hide-default-footer
  )
    template(
      v-slot:item="{item}"
    )
      tr(
        align="center"
      )
        td(rowspan=2) {{ item.number }}
        template(
          v-for="(score, index) in item.scores"
        )
          td(
            :class="{ 'bg-yellow-lighten-4' : (item.mahjongNumber[index] === 1), 'bg-grey-lighten-4' : (item.mahjongNumber[index] === 2), 'bg-orange-lighten-3' : (item.mahjongNumber[index] === 3) }"
          )
            v-chip(
              :color="score.total > 0 ? 'success' : score.total < 0 ? 'error': ''"
            ) {{ score.total }}
      tr(
        align="center"
      )
        template(
          v-for="(score, index) in item.scores"
        )
          td(
            :class="{ 'bg-yellow-lighten-4' : (item.mahjongNumber[index] === 1), 'bg-grey-lighten-4' : (item.mahjongNumber[index] === 2), 'bg-orange-lighten-3' : (item.mahjongNumber[index] === 3) }"
          )
            template(
              v-for="(detail, index) in score.detail"
            )
              v-chip(
                :color="detail.score > 0 ? 'success' : 'error'"
              ) {{ Math.abs(detail.score) }}
        <!--td {{ item  }}-->
  v-btn(
    class="mt-4"
    color="error"
    block
    @click="reset"
  ) Прервать игру

v-dialog(
  width="auto"
  v-model="displaySettings.mahjong"
)
  v-card
    v-card-title Маджонг {{ gameState.mahjongCount + 1 }}
    v-card-subtitle Кто объявил маджонг?
    v-card-text
      v-form
        v-radio-group(
          v-model="displaySettings.mahjongPlayer"
        )
          template(
            v-for="item in displaySettings.items"
          )
            v-radio(
              :label="item.key"
              :value="item.value"
            )
    v-card-actions
      v-btn(
        text="Ok"
        color="success"
        variant="tonal"
        :disabled="displaySettings.mahjongPlayer === -1"
        @click="mahjong"
      )
      v-btn(
        text="Отмена"
        color="error"
        variant="tonal"
        @click="displaySettings.mahjong = false"
      )

v-dialog(
  width="auto"
  min-width=300
  v-model="displaySettings.kong"
)
  v-card
    v-card-title Конг
    v-card-text
      v-form
        v-select(
          label="Кто объявил конг?"
          :items="displaySettings.items"
          item-title="key"
          item-value="value"
          v-model="displaySettings.kongPlayer"
          @update:modelValue="updateKongSourceList"
        )
        v-radio-group(
          v-model="displaySettings.kongFrom"
          v-if="displaySettings.kongPlayer !== null"
        )
          v-radio(label="Закрытый конг" value=-2)
          v-radio(label="Доставленный конг" value=-1)
          template(
            v-for="item in displaySettings.kongItems"
          )
            v-radio(
              :value="item.value"
              :label="'с ' + item.key"
            )
    v-card-actions
      v-btn(
        text="Ok"
        color="success"
        variant="tonal"
        :disabled="displaySettings.kongFrom === 0"
        @click="kong"
      )
      v-btn(
        text="Отмена"
        color="error"
        variant="tonal"
        @click="displaySettings.kong = false"
      )
</template>

<script>
  export default {
    data: () => ({
      gameState: {
        players: [ '', '', '', '' ],
        rounds: [],
        mahjongCount: 0,
        endOfWall: false
      },
      displaySettings: {
        needInitPlayer: true,
        roundInProggress: false,
        mahjong: false,
        mahjongPlayer: -1,
        kong: false,
        kongPlayer: null,
        kongFrom: 0,
        items: [],
        kongItems: [],
        headers: []
      },
      formInitPlayers: {
        players: [ 'Восток', 'Юг', 'Запад', 'Север'],
        random: false
      }
    }),
    methods: {
      initPlayers () {
        this.gameState = {
          players: [],
          rounds: [],
          mahjongCount: 0,
          endOfWall: false
        }
        if (this.formInitPlayers.random) {
          const temp = this.formInitPlayers.players.sort(() => Math.random() - 0.5).sort(() => Math.random() - 0.5).sort(() => Math.random() - 0.5).sort(() => Math.random() - 0.5).sort(() => Math.random() - 0.5)
          this.gameState.players.push(...temp)
        } else {
          this.gameState.players.push(...this.formInitPlayers.players)
        }
        this.displaySettings.headers = [
          { title: "Раунд", key: "number" },
          { title: "東 Восток", align: 'center', children: [ { title: this.gameState.players[0], align: 'center', key: 'scores[0].total' } ] },
          { title: "南 Юг", align: 'center', children: [ { title: this.gameState.players[1], align: 'center', key: 'scores[1].total' } ] },
          { title: "西 Запад", align: 'center', children: [ { title: this.gameState.players[2], align: 'center', key: 'scores[2].total' } ] },
          { title: "北 Север", align: 'center', children: [ { title: this.gameState.players[3], align: 'center', key: 'scores[3].total' } ] }
        ]
        this.newRound()
        this.displaySettings.needInitPlayer = false
      },
      newRound () {
        const item = {
          number: this.gameState.rounds.length + 1,
          scores: [],
          inGame: [ true, true, true, true ],
          mahjongNumber: [ -1, -1, -1, -1 ],
          events: []
        }
        for (let i = 0; i < 4; i++) {
          item.scores.push({
            total: 0,
            detail: []
          })
        }
        this.gameState.rounds.unshift(item)
        this.gameState.mahjongCount = 0
        this.displaySettings.roundInProggress = true
      },
      reset () {
        this.formInitPlayers.players = [ 'Восток', 'Юг', 'Запад', 'Север']
        this.formInitPlayers.random = false
        this.displaySettings.needInitPlayer = true
      },
      getActivePlayers(excludePlayer) {
        const currentRound = this.gameState.rounds[0]
        const items = []
        for (let key in this.gameState.players) {
          if (currentRound.inGame[key] && (key !== excludePlayer)) {
            items.push(
              {
                key: this.gameState.players[key],
                value: key
              }
            )
          }
        }
        return items
      },
      displayMahjongDialog() {
        this.displaySettings.items = this.getActivePlayers()
        this.displaySettings.mahjongPlayer = -1
        this.displaySettings.mahjong = true
      },
      displayKongDialog() {
        this.displaySettings.items = []
        this.displaySettings.items = this.getActivePlayers()
        this.displaySettings.kongPlayer = null
        this.displaySettings.kong = true
        this.displaySettings.kongFrom = 0
        this.displaySettings.kongItems = []
      },
      updateKongSourceList() {
        this.displaySettings.kongItems = this.getActivePlayers(this.displaySettings.kongPlayer)
      },
      mahjong () {
        this.displaySettings.mahjong = false
        this.gameState.mahjongCount++
        const winner = this.displaySettings.mahjongPlayer
        const currentRound = this.gameState.rounds[0]
        currentRound.inGame[winner] = false
        currentRound.mahjongNumber[winner] = this.gameState.mahjongCount
        if (this.gameState.mahjongCount === 3) {
          this.endOfRound()
        }
      },
      kong () {
        this.displaySettings.kong = false
        const winner = this.displaySettings.kongPlayer
        const currentRound = this.gameState.rounds[0]
        const kongType = 1 * this.displaySettings.kongFrom
        if (kongType < 0) {
          const type = kongType === -2 ? 'concealed kong' : 'melded kong'
          const price = -kongType
          const loosers = this.getActivePlayers(winner)
          currentRound.scores[winner].total += price * loosers.length
          currentRound.scores[winner].detail.push({
             score: price * loosers.length,
             type: type,
             source: loosers
          })
          for (let looser of loosers) {
            currentRound.scores[looser.value].total -= price
            currentRound.scores[looser.value].detail.push({
              score: -price,
              type: type,
              destination: winner
            })
          }
        } else {
          currentRound.scores[winner].total += 2
          currentRound.scores[winner].detail.push({
             score: 2,
             type: 'kong',
             source: kongType
          })
          currentRound.scores[kongType].total -= 2
          currentRound.scores[kongType].detail.push({
            score: -2,
            type: 'kong',
            destination: winner
          })
        }
      },
      endOfWall () {
        this.gameState.endOfWall = true
        this.endOfRound()
      },
      endOfRound() {
        this.displaySettings.roundInProggress = false
        this.newRound()
      }
    }
  }
</script>
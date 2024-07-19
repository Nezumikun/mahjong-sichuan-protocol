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
      @click="reset"
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
        td {{ item.number }}
        td {{ item.scores[0].total }}
        td {{ item.scores[1].total }}
        td {{ item.scores[2].total }}
        td {{ item.scores[3].total }}
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
        items: [],
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
          console.log(temp)
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
          mahjongNumber: [ -1, -1, -1, -1 ]
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
      displayMahjongDialog() {
        const currentRound = this.gameState.rounds.at(0)
        this.displaySettings.items = []
        for (let key in this.gameState.players) {
          if (currentRound.inGame[key]) {
            this.displaySettings.items.push(
              {
                key: this.gameState.players[key],
                value: key
              }
            )
          }
        }
        this.displaySettings.mahjongPlayer = -1
        this.displaySettings.mahjong = true
      },
      mahjong () {
        this.displaySettings.mahjong = false
        this.gameState.mahjongCount++
        const winner = this.displaySettings.mahjongPlayer
        const currentRound = this.gameState.rounds.at(0)
        currentRound.inGame[winner] = false
        currentRound.mahjongNumber[winner] = this.gameState.mahjongCount
        if (this.gameState.mahjongCount === 3) {
          this.displaySettings.roundInProggress = false
          this.newRound()
        }
      },
      endOfWall () {
        this.gameState.endOfWall = true
        this.displaySettings.roundInProggress = false
        this.newRound()
      }
    }
  }
</script>
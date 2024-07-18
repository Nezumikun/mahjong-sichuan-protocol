<template lang="pug">
v-sheet(
  class="mx-auto"
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
  v-data-table(
    :headers="displaySettings.headers"
    :items="items"
    item-key="name"
    items-per-page="-1"
    hide-default-footer
  )
  v-btn(
    class="mt-4"
    color="error"
    block
    @click="reset"
  ) Отмена
</template>

<script>
  export default {
    data: () => ({
      gameState: {
        players: [ '', '', '', '' ]
      },
      displaySettings: {
        needInitPlayer: true,
        headers: []
      },
      formInitPlayers: {
        players: [ 'Восток', 'Юг', 'Запад', 'Север'],
        random: false
      },
      items: []
    }),
    methods: {
      initPlayers () {
        this.gameState.players = []
        if (this.formInitPlayers.random) {
          const temp = this.formInitPlayers.players.sort(() => Math.random() - 0.5).sort(() => Math.random() - 0.5).sort(() => Math.random() - 0.5).sort(() => Math.random() - 0.5).sort(() => Math.random() - 0.5)
          console.log(temp)
          this.gameState.players.push(...temp)
        } else {
          this.gameState.players.push(...this.formInitPlayers.players)
        }
        this.displaySettings.headers = [
          { title: "Раунд" },
          { title: "東 Восток", align: 'center', children: [ { title: this.gameState.players[0], align: 'center' } ] },
          { title: "南 Юг", align: 'center', children: [ { title: this.gameState.players[1], align: 'center' } ] },
          { title: "西 Запад", align: 'center', children: [ { title: this.gameState.players[2], align: 'center' } ] },
          { title: "北 Север", align: 'center', children: [ { title: this.gameState.players[3], align: 'center' } ] }
        ]        
        this.displaySettings.needInitPlayer = false
      },
      reset () {
        this.formInitPlayers.players = [ 'Восток', 'Юг', 'Запад', 'Север']
        this.formInitPlayers.random = false
        this.displaySettings.needInitPlayer = true
      }
    }
  }
</script>
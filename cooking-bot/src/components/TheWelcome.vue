<template>
  <h1>Macdonald Order System</h1>
  <div class="buttons">
    <button @click="orderNormal">New Normal Order</button>
    <button @click="orderVIP">New VIP Order</button>
    <button @click="addBot">+ Bot</button>
    <button @click="minusBot">- Bot</button>
  </div>
  <div class="order-status">
    <h2>Order Status</h2>
    <div class="status-grid">
      <div><strong>Pending:</strong> {{ pending.length }}</div>
      <div><strong>Complete:</strong> {{ complete.length }}</div>
      <div><strong>Total Bots:</strong> {{ bots.length }}</div>
    </div>
    <div class="bot-list">
      <h3>Bots</h3>
      <div v-if="bots.length">
        <p v-for="(bot, index) in bots" :key="index">{{ bot.name }} - <span :class="bot.status">{{ bot.status }}</span></p>
      </div>
      <p v-else>No bots active</p>
    </div>
  </div>
  <div class="order-lists">
    <div class="order-column">
      <h2>Pending List</h2>
      <div v-if="pending.length > 0" class="order-box">
        <p v-for="order in pending" :key="order.id">{{ order.id }} {{ order.type }} - {{ order.time_left }} <span v-if="order.bot.index !== null"> ({{ order.bot.name }})</span></p>
      </div>
      <p v-else class="no-orders">No pending orders</p>
    </div>

    <div class="order-column">
      <h2>Complete List</h2>
      <div v-if="complete.length > 0" class="order-box">
        <p v-for="order in complete" :key="order.id">{{ order.id }} {{ order.type }}</p>
      </div>
      <p v-else class="no-orders">No complete orders</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      orders: 0,
      bots: [],
      pending: [],
      complete: [],
      timer: null,
    }
  },
  created() {
    this.timer = setInterval(() => {
      this.updatePendingOrders()
    }, 1000)
  },
  beforeUnmount() {
    clearInterval(this.timer)
  },
  methods: {
    orderNormal() {
      this.orders += 1
      this.pending.push({
        id: this.orders,
        type: 'Normal',
        bot: {
          index: null,
          name: null
        },
        time_left: 10
      })
    },
    orderVIP() {
      this.orders += 1
      let numberVIP = this.pending.filter(order => order.type === 'VIP').length
      // VIP order are adding to the end of VIP list
      this.pending.splice(numberVIP, 0, {
        id: this.orders,
        type: 'VIP',
        bot: {
          index: null,
          name: null
        },
        time_left: 10
      })
    },
    addBot() {
      this.bots.push({
        name: `Bot ${this.bots.length + 1}`,
        status: 'idle'
      })
    },
    minusBot() {
      // check if any pending order using any bot
      const botOrder = this.pending.findIndex(order => order.bot.index === this.bots.length - 1)
      if (botOrder !== -1) {
        this.pending[botOrder].bot.index = null
        this.pending[botOrder].bot.name = null
        this.pending[botOrder].time_left = this.pending[botOrder].time_left
      }
      this.bots.splice(this.bots.length - 1, 1)
    },
    updatePendingOrders() {
      // skip if no pending order or bot
      if (this.pending.length === 0 || this.bots.length === 0) {
        return
      }

      this.pending.forEach((order, index) => {
        // if order not assiged to bot yet, assigned a bot if available
        if (order.bot.index === null) {
          const availableBotIndex = this.bots.findIndex(bot => bot.status === 'idle')
          if (availableBotIndex !== -1) {
            this.bots[availableBotIndex].status = 'busy'
            order.bot.index = availableBotIndex
            order.bot.name = this.bots[availableBotIndex].name
          } else if (availableBotIndex === -1 && order.type === 'VIP') { // if not idle bot then order type is VIP then check normal order bot avaibility
            const normalOrderIndex = this.pending.findIndex(order => order.type === 'Normal' && order.bot.index !== null);
            if (normalOrderIndex !== -1) {
              order.bot.index = this.pending[normalOrderIndex].bot.index
              order.bot.name = this.pending[normalOrderIndex].bot.name
              this.pending[normalOrderIndex].bot.index = null
              this.pending[normalOrderIndex].bot.name = null
            }
          }
        }

        // if order already have bot assiged then start countdown
        if (order.bot.index !== null) {
          order.time_left -= 1
        }

        // if order time is up then move to complete list
        if (order.time_left <= 0) {
          const completedOrder = this.pending.splice(index, 1)[0]
          this.complete.push(completedOrder)

          console.log('time', this.bots)

          this.bots[order.bot.index].status = 'idle'
        }
      })
    },
  }
}
</script>
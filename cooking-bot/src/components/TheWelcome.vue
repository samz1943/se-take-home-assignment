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
        <p v-for="order in pending" :key="order.id">{{ order.id }} {{ order.type }} - {{ order.time_left }}</p>
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
        bot_index: null,
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
        bot_index: null,
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
      const botOrder = this.pending.findIndex(order => order.bot_index === this.bots.length - 1)
      if (botOrder !== -1) {
        this.pending[botOrder].bot_index = null
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
        if (order.bot_index === null) {
          const availableBotIndex = this.bots.findIndex(bot => bot.status === 'idle')
          if (availableBotIndex !== -1) {
            order.bot_index = availableBotIndex
            this.bots[availableBotIndex].status = 'busy'
          } else if (availableBotIndex === -1 && order.type === 'VIP') { // if not idle bot then order type is VIP then check normal order bot avaibility
            const normalOrderIndex = this.pending.findIndex(order => order.type === 'Normal' && order.bot_index !== null);
            if (normalOrderIndex !== -1) {
              order.bot_index = this.pending[normalOrderIndex].bot_index
              this.pending[normalOrderIndex].bot_index = null
            }
          }
        }

        // if order already have bot assiged then start countdown
        if (order.bot_index !== null) {
          order.time_left -= 1
        }

        // if order time is up then move to complete list
        if (order.time_left <= 0) {
          const completedOrder = this.pending.splice(index, 1)[0]
          this.complete.push(completedOrder)

          this.bots[order.bot_index].status = 'idle'
        }
      })
    },
  }
}
</script>

<style scoped>
h1, h2 {
  color: #2c3e50;
  margin-bottom: 20px;
}

h2 {
  font-size: 1.5rem;
  margin-top: 30px;
}

.buttons {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
  transition: background-color 0.3s;
}

.buttons button:nth-child(1) {
  background-color: #42b983;
  color: white;
}

.buttons button:nth-child(2) {
  background-color: #e74c3c;
  color: white;
}

.buttons button:nth-child(3), .buttons button:nth-child(4) {
  background-color: #3498db;
  color: white;
}

button:hover {
  opacity: 0.8;
}

button.active {
  box-shadow: 0 0 0 2px white, 0 0 0 4px currentColor;
}

.order-status {
  background-color: #fdfdfd;
  padding: 20px;
  border-radius: 6px;
  margin-bottom: 30px;
}

.status-grid {
  display: flex;
  gap: 40px;
  font-size: 1.1rem;
  margin-bottom: 15px;
}

.bot-list span.idle {
  color: green;
}
.bot-list span.busy {
  color: red;
}

.orders-section {
  margin-top: 30px;
}

.order-lists {
  display: flex;
  gap: 40px;
  justify-content: space-between;
  margin-top: 30px;
}

.order-column {
  flex: 1;
}

.order-box {
  background-color: #f4f6f8;
  border-radius: 5px;
  padding: 15px;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.05);
  min-height: 150px;
}

.no-orders {
  padding: 15px;
  background-color: #f0f0f0;
  border-radius: 4px;
  text-align: center;
  color: #999;
  margin-top: 10px;
}
</style>
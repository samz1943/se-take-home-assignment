<template>
  <h1>Macdonald Order System</h1>
  <div class="buttons">
    <button @click="orderNormal">New Normal Order</button>
    <button @click="orderVIP">New VIP Order</button>
    <button @click="addBot">+ Bot</button>
    <button @click="minusBot">- Bot</button>
  </div>
  <div>
    <h2>Order Status</h2>
    <div>
      <p>Total Pending Orders: {{ pending.length }}</p>
      <p>Total Complete Orders: {{ complete.length }}</p>
      <p>Total Bots:</p>
      <div>
        <p v-for="bot in bots" :key="bot.index">{{ bot.name }} - {{ bot.status }}</p>
      </div>
    </div>
  </div>
  <div>
    <h2>Pending List</h2>
    <div>
      <p v-for="order in pending" :key="order.id">{{ order.id }} {{ order.type }} - {{ order.time_left }}</p>
    </div>
    <h2>Complete List</h2>
    <div>
      <p v-for="order in complete" :key="order.id">{{ order.id }} {{ order.type }}</p>
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
      const botOrder = this.pending.findIndex(order => order.bot_index === this.bots.length - 1)
      if (botOrder !== -1) {
        this.pending[botOrder].bot_index = null
        this.pending[botOrder].time_left = this.pending[botOrder].time_left
      }
      this.bots.splice(this.bots.length - 1, 1)
    },
    updatePendingOrders() {
      if (this.pending.length === 0 || this.bots.length === 0) {
        return
      }

      this.pending.forEach((order, index) => {
        if (order.bot_index === null) {
          const availableBotIndex = this.bots.findIndex(bot => bot.status === 'idle')
          if (availableBotIndex !== -1) {
            order.bot_index = availableBotIndex
            this.bots[availableBotIndex].status = 'busy'
          } else if (availableBotIndex === -1 && order.type === 'VIP') {
            const normalOrderIndex = this.pending.findIndex(order => order.type === 'Normal' && order.bot_index !== null);
            if (normalOrderIndex !== -1) {
              order.bot_index = this.pending[normalOrderIndex].bot_index
              this.pending[normalOrderIndex].bot_index = null
            }
          }
        }

        if (order.bot_index !== null) {
          order.time_left -= 1
        }
        
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

.orders-section {
  margin-top: 30px;
}

.order-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 15px;
}

.order-card {
  background-color: #f8f9fa;
  border-radius: 4px;
  padding: 15px;
  border-left: 4px solid #42b983;
  position: relative;
}

.order-card.completed {
  border-left-color: #3498db;
  opacity: 0.8;
}

.order-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
  font-weight: bold;
}

.order-type {
  padding: 3px 8px;
  border-radius: 3px;
  color: white;
  font-size: 0.9rem;
}

.order-type.normal {
  background-color: #42b983;
}

.order-type.vip {
  background-color: #e74c3c;
}

.order-time {
  color: #6c757d;
  font-size: 0.9rem;
}

.order-details {
  margin-bottom: 15px;
}

.complete-button {
  background-color: #3498db;
  color: white;
  padding: 5px 10px;
  font-size: 0.9rem;
}

.delivery-note {
  margin-top: 5px;
  font-style: italic;
  color: #6c757d;
}

.delivery-note small {
  display: block;
  margin-bottom: 10px;
}

.no-orders {
  margin-top: 30px;
  padding: 20px;
  background-color: #f8f9fa;
  border-radius: 4px;
  text-align: center;
  color: #6c757d;
}
</style>
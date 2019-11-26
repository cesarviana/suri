<template>
  <div class="container">
    <div>
      <h1>SURI - Ambiente de simulação</h1>
      <span>{{currentTime}}</span>
      <canvas id="myCanvas" ref="canvas" :width="canvas.width" :height="canvas.height" style="border:1px solid #000000;"></canvas>

    </div>
  </div>
</template>

<script>
import Logo from '~/components/Logo.vue'
import jssim from 'js-simulator'

class Vehycle extends jssim.SimEvent {
  constructor(id, x, y, space){
    super()
    const rank = 1
    jssim.SimEvent.call(this, rank)
    this.id = id
    this.space = space
    this.space.updateAgent(this, x, y)
    this.velocity = new jssim.Vector2D(3, 0)

    this.border = 30
    this.color = 'blue'
    this.size = new jssim.Vector2D(10, 10)
  }
  update() {
    const pos = this.space.getLocation(this.id)
    pos.x += this.velocity.x

    const neighbors = this.space.getNeighborsWithinDistance(pos, 30)
    const emergency = neighbors.find(agent=>agent instanceof Emergency)
    if(emergency) {
      this.color = 'red'
      this.velocity.x = -1
    }

  }
}

class Emergency extends jssim.SimEvent {
  constructor(id, x, y, space) {
    super()
    this.x = x
    this.y = y
    const rank = 1
    jssim.SimEvent.call(this, rank)
    this.id = id
    this.space = space
  }
  update() {
    this.space.updateAgent(this, this.x, this.y)
    this.color = 'red'
    this.border = 30
    this.size = new jssim.Vector2D(20, 20)
  }
}

export default {
  components: {
    Logo
  },
  data() {
    return {
      maxVehycles: 30,
      canvas: {
        width: 640,
        height: 640
      },
      currentTime: 0
    }
  },
  mounted() {
    this.startSimulation()
  },
  methods: {
    startSimulation() {
      
      const scheduler = new jssim.Scheduler()
      scheduler.reset()

      const grid = new jssim.Grid(this.canvas.width, this.canvas.height)

      const space = new jssim.Space2D()
      space.reset()

      const xRoad = 0
      const yRoad = this.canvas.height / 2
      const xEndRoad = this.canvas.width
      const yEndRoad = yRoad
      const roadWidth = 40
      space.drawLine(xRoad, yRoad, xEndRoad, yEndRoad, { stroke: 'gray', width: roadWidth })

      for(let i=0; i<this.maxVehycles; i++){
        const x = Math.ceil(Math.random() * this.canvas.width)
        const y = Math.ceil(Math.random() * roadWidth / 2) + yRoad
        const vehycle = new Vehycle(i, x, y, space)
        scheduler.scheduleRepeatingIn(vehycle, 10)
      }

      const xEmergency = Math.random() * this.canvas.width / 2 + this.canvas.width / 2
      const yEmergency = Math.ceil(Math.random() * roadWidth / 2) + yRoad

      const emergency = new Emergency(-1, xEmergency, yEmergency, space)
      scheduler.schedule(emergency, 50)

      const canvas = this.$refs.canvas
      setInterval(()=>{
        scheduler.update()
        space.render(canvas)
        this.currentTime = scheduler.current_time
      }, 40)
    }
  }
}
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>

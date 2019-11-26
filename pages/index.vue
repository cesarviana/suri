<template>
  <div class="container">
    <div>
      <h1>SURI - Ambiente de simulação</h1>
      <canvas id="myCanvas" ref="canvas" :width="canvas.width" :height="canvas.height" style="border:1px solid #000000;"></canvas>
      <br>
      <div>
        <b>Número de veículos</b>
        {{numVehycles}}
        <input type="range" v-model="numVehycles">
      </div>
      <div>
        <b>Tempo para atualização</b>
        {{timeToUpdate}}
        <input type="range" v-model="timeToUpdate" min="100" max="1000">
      </div>
      <div>
        <button @click="restart">Reiniciar</button>
      </div>

    </div>
  </div>
</template>

<script>
import Logo from '~/components/Logo.vue'
import jssim from 'js-simulator'

class Vehycle extends jssim.SimEvent {
  constructor(id, x, y, space, roads){
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
    this.roads = roads
  }
  update() {
    const pos = this.space.getLocation(this.id)
    this.runOverRoads(pos)

    if(this.emergency){
      this.handleEmergency(this.emergency)
    }

    const messages = this.readInBox(this)
    if(messages.length > 0) {
      this.emergency = messages[0].content
      this.handleEmergency(this.emergency)
    }  

    const neighbors = this.space.getNeighborsWithinDistance(pos, 30)
    const emergency = neighbors.find(agent=>agent instanceof Emergency)
    if(emergency) {
      this.emergency = emergency
      this.handleEmergency(emergency)
    }
  }

  runOverRoads(pos) {
    pos.x += this.velocity.x
    pos.y += this.velocity.y
  }

  handleEmergency(emergency) {
    this.color = 'red'
    this.escape(emergency)
    this.notifyEmergencyToNeighbors(emergency)
  }

  notifyEmergencyToNeighbors(emergency) {
    const pos = this.space.getLocation(this.id)
    this.space.getNeighborsWithinDistance(pos, 30)
    .filter(agent => agent instanceof Vehycle)
    .filter(vehicle => vehicle.emergency == undefined)
    .forEach(vehycle => this.sendMsg(vehycle.id, {
      content: emergency
    }))
  }

  escape(emergency) {
    const thisPos = this.space.getLocation(this.id)
    const emergencyPos = this.space.getLocation(emergency.id)
    const forwarding = this.velocity.x > 0
    const wasBlocked = emergencyPos.x > thisPos.x
    if(!this.alternativeRoute && forwarding && wasBlocked){
      this.velocity.x = -this.velocity.x
    }
    if(thisPos.x < 50){
      this.alternativeRoute = true
      this.color = 'brown'
      this.velocity.x = Math.abs(this.velocity.x)
      this.velocity.y = -0.8
      this.emergency = undefined
    }
    if(thisPos.y < 20) {
      this.velocity.y = 0.8
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
    this.velocity = new jssim.Vector2D(0, 0)
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
      numVehycles: 40,
      timeToUpdate: 100,
      handleInterval: undefined, 
      canvas: {
        width: 1200,
        height: 300
      },
      currentTime: 0,
      scheduler: new jssim.Scheduler(),
      space: new jssim.Space2D()
    }
  },
  methods: {
    startSimulation() {
      clearInterval(this.handleInterval)
      this.scheduler.reset()
      this.space.reset()
      
      const road = {
        x: 0,
        y: this.canvas.height / 2,
        endX: this.canvas.width,
        endY: this.canvas.height / 2,
        width: 60
      }

      const roads = [
        road, 
        {
          x: 0,
          y: this.canvas.height / 2,
          endX: this.canvas.width / 2,
          endY: 0,
          width: 60
        },
        {
          x: this.canvas.width / 2 - 10,
          y: 0,
          endX: this.canvas.width,
          endY: this.canvas.height / 2,
          width: 60
        }
      ]

      roads.forEach(road => this.space.drawLine(road.x, road.y, road.endX, road.endY, { stroke: 'gray', width: road.width }))
      
      for(let i=0; i<this.numVehycles; i++){
        const x = Math.ceil(Math.random() * this.canvas.width)
        const y = Math.ceil(Math.random() * road.width / 2 ) + road.y
        // const direction = y > road.
        const vehycle = new Vehycle(i, x, y, this.space, roads)
        this.scheduler.scheduleRepeatingIn(vehycle, this.timeToUpdate)
      }

      const scheduleTime = 50
      this.scheduleEmergency(this.scheduler, this.space, road, scheduleTime)

      const canvas = this.$refs.canvas
      this.handleInterval = setInterval(()=>{
        this.scheduler.update()
        this.space.render(canvas)
        this.currentTime = this.scheduler.current_time
      }, this.timeToUpdate)
    },
    scheduleEmergency(scheduler, space, road, scheduleTime) {
      const xEmergency = road.endX / 2
      const yEmergency = road.y + 10
      const emergency = new Emergency(jssim.guid(), xEmergency, yEmergency, space)
      scheduler.schedule(emergency, scheduleTime)
    },
    restart() {
      this.startSimulation()
    }
  }
}
</script>

<style>
.container {
  background: lightgreen;
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

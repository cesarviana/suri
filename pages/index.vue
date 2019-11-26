<template>
  <div class="container">
    <div>
      <h1>SURI - Ambiente de simulação</h1>

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
    this.velocity = new jssim.Vector2D(0.5, 0.5)

    this.border = 30
    this.color = 'red'
    this.size = new jssim.Vector2D(10, 10)
  }
  update() {
    const pos = this.space.getLocation(this.id)
    pos.x = 1
    pos.y = 150
  }
}

export default {
  components: {
    Logo
  },
  data() {
    return {
      maxVehycles: 10,
      canvas: {
        width: 640,
        height: 640
      }
    }
  },
  mounted() {
    this.startSimulation()
  },
  methods: {
    startSimulation() {
      
      const scheduler = new jssim.Scheduler()
      scheduler.reset()

      const space = new jssim.Space2D()
      space.reset()

      for(let i=0; i<this.maxVehycles; i++){
        const x = Math.ceil(Math.random() * this.canvas.width)
        const y = Math.ceil(Math.random() * this.canvas.height)
        console.log(x, y)
        const vehycle = new Vehycle(i, x, y, space)
        scheduler.scheduleRepeatingIn(vehycle, 1)
      }

      const canvas = this.$refs.canvas
      setInterval(()=>{
        scheduler.update()
        space.render(canvas)
      }, 100)
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

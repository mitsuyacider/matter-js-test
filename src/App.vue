<template>
  <div id="app">
    <canvas id="canvas" class="canvas" />
  </div>
</template>

<style>
body {
  margin: 0;
}
#app {
  background: white;
  width: 100vw;
  height: 100vh;
}
canvas {
  width: 100px;
  height: 100vh;
}
</style>

<script>
import HelloWorld from './components/HelloWorld'
// import './js/matterTest.js'
import Matter from 'matter-js'

export default {
  name: 'App',
  components: {
    HelloWorld
  },
  data () {
    return {
      engine: {},
      Body: {},
      boxC: {}
    }
  },
  mounted () {
    // NOTE: キャンバスサイズはdefaultで300px。attr属性もしくは
    //       直接JavaScriptで指定する必要がある
    // https://qiita.com/ShinyaOkazawa/items/9e662bf2121548f79d5f
    var canvas = document.getElementById('canvas')
    canvas.width = screen.width * 2
    canvas.height = screen.height * 2
    canvas.style.width = String(canvas.width / 2) + 'px'
    canvas.style.height = String(canvas.height / 2) + 'px'

    // module aliases
    let Engine = Matter.Engine
    let World = Matter.World
    let Bodies = Matter.Bodies
    this.Body = Matter.Body
    let Events = Matter.Events
    // create an engine
    this.engine = Engine.create()

    // create two boxes and a ground
    let boxA = Bodies.rectangle(600, 200, 80, 80, { friction: 0.01, restitution: 0.1, density: 1.5 })
    let boxB = Bodies.rectangle(250, 250, 180, 80, { friction: 0, restitution: 0.1 })
    this.boxC = Bodies.rectangle(400,
      0,
      200,
      100,
      { restitution: 1.95,
        friction: 0,
        render: {
          fillStyle: '#FFFFFF',
          text: {
            fillStyle: '#000000',
            content: 'Test TEST',
            size: 50
          }
        }
      })

    let ground = Bodies.rectangle(373, 840, 373 * 2, 120, { isStatic: true })
    World.add(this.engine.world, [boxA, boxB, ground, this.boxC])
    Events.on(this.engine, 'beforeUpdate', this.matterBeforeUpdate)
    // run the engine
    Engine.run(this.engine)

    // ------ NOTE: This is default render setting
    // let Render = Matter.Render
    // create a renderer
    // let render = Render.create({
    //   element: document.getElementById('app'),
    //   engine: this.engine,
    //   options: {
    //     wireframes: false, // ワイヤーフレームモードをoff
    //     width: 300, // canvasのwidth(横幅)
    //     height: 400, // canvasのheight(高さ)
    //     background: 'rgba(255, 0, 0, 0.5)'
    //   }
    // })
    // Render.run(render)
    // ------
    this.render()
  },
  methods: {
    matterBeforeUpdate (event) {
      this.Body.setAngularVelocity(this.boxC, 0)
    },
    render () {
      var bodies = Matter.Composite.allBodies(this.engine.world)
      var canvas = document.getElementById('canvas')
      var context = canvas.getContext('2d')

      window.requestAnimationFrame(this.render)

      context.fillStyle = '#ff0000'
      context.fillRect(0, 0, canvas.width, canvas.height)
      context.globalAlpha = 1
      context.beginPath()

      for (var i = 0; i < bodies.length; i += 1) {
        var part = bodies[i]

        if (part.render.text) {
          // 30px is default font size
          var fontsize = 30
          // arial is default font family
          var fontfamily = part.render.text.family || 'Arial'
          // white text color by default
          var color = part.render.text.color || '#FFFFFF'

          if (part.render.text.size) {
            fontsize = part.render.text.size
          } else if (part.circleRadius) {
            fontsize = part.circleRadius / 2
          }

          var content = ''
          if (typeof part.render.text === 'string') {
            content = part.render.text
          } else if (part.render.text.content) {
            content = part.render.text.content
          }

          context.fillStyle = 'black'
          context.save()
          context.translate(part.position.x, part.position.y)

          const x = bodies[i].vertices[1].x - bodies[i].vertices[0].x
          const y = bodies[i].vertices[1].y - bodies[i].vertices[0].y
          const radian = Math.atan2(y, x)
          context.rotate(radian)
          context.textBaseline = 'middle'
          context.textAlign = 'center'
          context.fillStyle = color
          context.font = fontsize + 'px ' + fontfamily
          context.fillText(content, 0, 0)
          context.restore()
          context.fillStyle = 'blue'
          context.fillRect(part.position.x, part.position.y, 10, 10)
        }
        var vertices = bodies[i].vertices
        context.moveTo(vertices[0].x, vertices[0].y)

        for (var j = 1; j < vertices.length; j += 1) {
          context.lineTo(vertices[j].x, vertices[j].y)
        }

        context.lineTo(vertices[0].x, vertices[0].y)
      }

      context.lineWidth = 1.5
      context.strokeStyle = '#000000'
      context.stroke()
    }
  }
}
</script>

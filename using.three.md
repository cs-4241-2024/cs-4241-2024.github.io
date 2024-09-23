# Three.js
This example illustrates using [three.js](threejs.org), with a quick example of [tweakpane](https://cocopon.github.io/tweakpane/) to control it.

In three.js, we'll make a *scene*, which consists of geometries and lights. We then pick a rendering engine; WebGL is the most common
candidate, but you could also render to SVG or using the newer WebGPU API. We'll also create a camera. Once all these assets are in place,
we can tell our *renderer* to render a specific *scene* (geometries and lights) using a specific *camera*.

We'll keep our html file simple:

```html
<!doctype html>
<html lang='en'>
  <head>
    <style> body { margin:0 } </style>
    <script src='./main.js' type="module"></script>
  </head>
  <body></body>
</html>

```

Our `main.js` script will use JS modules from the [unpkg CDN](https://unpkg.com). In order to view the page properly you'll
need to run it with a web server. [http-server](https://github.com/http-party/http-server) is a simple choice for this. Run
the following command to install and start it: `npx http-server .`

Now create the JS file below, and then open up `http://127.0.0.1:8080` in your browser. `8080` is the default port used
by http-server.

```js
// import our three.js reference
import * as THREE from 'https://unpkg.com/three/build/three.module.js'
import { Pane } from 'https://unpkg.com/tweakpane'

const app = {
  init() {
    this.scene = new THREE.Scene()

    this.camera = new THREE.PerspectiveCamera( 45, window.innerWidth/window.innerHeight, 1, 1000 )
    this.camera.position.z = 50 

    this.renderer = new THREE.WebGLRenderer()
    this.renderer.setSize( window.innerWidth, window.innerHeight )

    document.body.appendChild( this.renderer.domElement )
    
    this.createLights()
    this.knot = this.createKnot()

    // ...the rare and elusive hard binding appears! but why?
    this.render = this.render.bind( this )
    this.render()

    // create a new tweakpane instance
    this.pane = new Pane()
    // setup our pane to control the know rotation on the y axis
    this.pane.addBinding( this.knot.rotation, 'y' )
  },

  createLights() {
    const pointLight = new THREE.DirectionalLight( 0xcccccc, 2 )  
    this.scene.add( pointLight )
  },

  createKnot() {
    const knotgeo = new THREE.TorusKnotGeometry( 10, .5, 128, 16, 5, 21 )
    const mat     = new THREE.MeshPhongMaterial({ color:0xff0000, shininess:2000 }) 
    const knot    = new THREE.Mesh( knotgeo, mat )

    this.scene.add( knot )
    return knot
  },

  render() {
    this.knot.rotation.x += .025
    this.renderer.render( this.scene, this.camera )
    window.requestAnimationFrame( this.render )
  }
}

window.onload = ()=> app.init()
```

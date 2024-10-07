<svelte:head>
  <title>Orrery</title>
</svelte:head>

<script>
import CameraControls from 'camera-controls';
import {
    Box3,
    BoxGeometry,
    BufferGeometry,
    Clock,
    DirectionalLight,
    EllipseCurve,
    GridHelper,
    HemisphereLight,
    Line,
    LineBasicMaterial,
    Matrix4,
    Mesh,
    MeshBasicMaterial,
    PerspectiveCamera,
    Quaternion,
    Raycaster,
    Scene,
    Sphere,
    Spherical,
    Vector2,
    Vector3,
    Vector4,
    WebGLRenderer,
    Layers,
    BufferAttribute,
    SphereGeometry
} from 'three';

import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass.js';
import { OutputPass } from 'three/addons/postprocessing/OutputPass.js';
import { degToRad } from 'three/src/math/MathUtils';

let canvas;

$effect(() => {


CameraControls.install({ THREE: {
  Vector2, Vector3, Vector4, Quaternion, Matrix4, Spherical, Box3, Sphere, Raycaster
}})

const scene = new Scene()
const clock = new Clock()
const camera = new PerspectiveCamera(75, innerWidth/innerHeight, 0.1, 10000)
camera.position.set( 0, 5, 5 );
camera.lookAt(0, 0, 1)
const controls = new CameraControls(camera, canvas)
const frame = new WebGLRenderer({ canvas, antialias: true })


const layer_bloom = new Layers()
layer_bloom.set(1)

const pass_render = new RenderPass(scene, camera)

const pass_bloom = new UnrealBloomPass(new Vector2(innerWidth, innerHeight), 1.5, 0.4, 0.85);
pass_bloom.threshold = 0;
pass_bloom.strength = 0.2;
pass_bloom.radius = 0;

const pass_output = new OutputPass()

const composer = new EffectComposer(frame)
composer.addPass(pass_render)
composer.addPass(pass_bloom)
composer.addPass(pass_output)
/*

Keplarian parameters spec

a: semi-major axis (au)
e: eccentricity
I: inclination (degrees/century)
L: mean longitude (deg/cent)
w: longitude of perihelion (deg/cent)
o: (raan) longitude of ascending node (deg/cent)

Epoch: something (MJD)

mean  
*/

const data = [
  {
    name: "mercury",
    axis: 1,
    eccent: 0.95,
    inclin: 0,
    long_mean: 252.25032350,
    long_peri: 102.9,
    long_asc_node: 0,
  },
]

let anomaly = 0
const RAD = Math.PI/180

function orbit_compute(data) {

  let {
    axis,
    eccent: e,
    inclin: i,
    long_mean,
    long_peri: w,
    long_asc_node: O
  } = data

  i = degToRad(i)
  w = degToRad(w)
  O = degToRad(O)
  const v = degToRad(anomaly)
  const r = (anomaly * (1 - e^2)) / (1 + e * Math.cos(v));

  const x = r * Math.cos(v)
  const y = r * Math.sin(v)

  return new Vector3(
    x*(Math.cos(w)*Math.cos(O) - Math.sin(w)*Math.sin(O)*Math.cos(i)) - y*(Math.sin(w)*Math.cos(O) + Math.cos(w)*Math.sin(O)*Math.cos(i)),
    x*(Math.cos(w)*Math.cos(O) + Math.sin(w)*Math.sin(O)*Math.cos(i)) - y*(Math.sin(w)*Math.cos(O) - Math.cos(w)*Math.sin(O)*Math.cos(i)),
    x*(Math.sin(w)*Math.sin(i)) + (Math.cos(w)*Math.sin(i))
  )
}

const epoch = Date.now()

const mesh_spheretest = new Mesh(
  new SphereGeometry(1, 32, 16),
  new MeshBasicMaterial({ color: 0xffff00 })
)

mesh_spheretest.position.set(10, 0, 0)

scene.add(mesh_spheretest)

const curve = new EllipseCurve(0, 0, 10, 10);
const colors = Array.from({length:40}, (v, i) => Array.from({length: 3}, v => (40-i)/40)).flat()
mesh_spheretest.layers.set(1)


console.log(colors)


const points = curve.getPoints(50);
let geo_orbit = new BufferGeometry().setFromPoints(points)
geo_orbit.setAttribute('color', new BufferAttribute(new Float32Array(colors), 3))
const orbit = new Line(
  geo_orbit,
  new LineBasicMaterial({ vertexColors: true })
);

orbit.layers.set(1)
orbit.rotation.x = degToRad(90)
scene.add(orbit)




const mesh_boxtest = new Mesh(
	new BoxGeometry( 1, 1, 1 ),
	new MeshBasicMaterial( { color: 0xff0000, wireframe: true } )
);
scene.add(mesh_boxtest);

const light_base = new HemisphereLight(0xffffff, 0x8d8d8d, 3)
scene.add(light_base)

const light_dir = new DirectionalLight(0xffffff, 3)
scene.add(light_dir)

const grid = new GridHelper(100, 10, 0x888888, 0x444444)
scene.add(grid)



function resize() {
  camera.aspect = innerWidth / innerHeight
  camera.updateProjectionMatrix()
  frame.setSize(innerWidth, innerHeight)
  composer.setSize(innerWidth, innerHeight);
}
addEventListener('resize', resize)
resize()

function draw() {
  const dt = clock.getDelta();
	controls.update(dt);

  // anomaly += 1

  // if(anomaly > 360) anomaly = 0

  // console.log(orbit_compute(data[0]).divide(new Vector3(2, 2, 2)))
  // mesh_spheretest.position.copy(orbit_compute(data[0]).divide(new Vector3(10, 10, 10)))
  
  frame.autoClear = false
  camera.layers.set(1)
  composer.render()

  camera.layers.set(0)
  frame.clearDepth()
  frame.render(scene, camera)
}
frame.setAnimationLoop(draw)

})

</script>

<canvas bind:this={canvas}></canvas>

<div class='fixed top-4 left-4'>
  <h1 class='text-gray-300 font-semibold'>Orrery (Incomplete)</h1>
  <p class='text-gray-500 max-w-[30ch]'>Unfortunately this was all I could complete because I had too much homework. Maybe next time?</p>
</div>

<style>
:global(html, body) {
  height: 100%;
  overflow: hidden;
}

canvas {
  width: 100%;
  height: 100%;
}

</style>
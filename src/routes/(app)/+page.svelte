<svelte:head>
  <title>Orrery</title>
</svelte:head>

<script>
import CameraControls from 'camera-controls';
import {
    Box3,
    BoxGeometry,
    CircleGeometry,
    Clock,
    DirectionalLight,
    GridHelper,
    HemisphereLight,
    LineLoop,
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
    SphereGeometry,
} from 'three';
let canvas;

$effect(() => {

CameraControls.install({ THREE: {
  Vector2, Vector3, Vector4, Quaternion, Matrix4, Spherical, Box3, Sphere, Raycaster
}})

const scene = new Scene()
const clock = new Clock()
const camera = new PerspectiveCamera(75, innerWidth/innerHeight, 0.1, 1000)
camera.position.set( 0, 5, 5 );
camera.lookAt(0, 0, 1)
const controls = new CameraControls(camera, canvas)
const frame = new WebGLRenderer({ canvas })

/*

Keplarian parameters spec

a: semi-major axis (au)
e: eccentricity
I: inclination (degrees/century)
L: mean longitude (deg/cent)
w: longitude of perihelion (deg/cent)
o: (raan) longitude of ascending node (deg/cent)

Semi-major axis (a): half the distance of the major axis (AU)
Eccentricity (e): shape of the ellipse
Inclination (i): vertical tilt (degrees)
mean longitude (l): something (degrees)
longitude of perhelion/perapsis (w): something
longitude of ascending node (ohm): something (degrees)

Epoch: something (MJD)

mean  
*/

const data = [
  {
    name: "mercury",
    axis: 0.38709927,
    eccent: 0.20563593,
    inclin: 7.00497902,
    long_mean: 252.25032350,
    long_peri: 77.45779628,
    long_asc_node: 48.33076593,
  },
]

function orbit_compute(data) {
  return new Vector3(
    
  )
}

const epoch = Date.now()
// const mesh_spheretest = new Mesh(
//   new SphereGeometry(15, 32, 16),
//   new MeshBasicMaterial({ color: 0xffff00 })
// )
// scene.add(mesh_spheretest)

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
}
addEventListener('resize', resize)
resize()

function draw() {
  const dt = clock.getDelta();
	controls.update(dt);
  
  frame.render(scene, camera)
}
frame.setAnimationLoop(draw)

})

</script>

<canvas bind:this={canvas}></canvas>

<div class='fixed top-1 left-1'>
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
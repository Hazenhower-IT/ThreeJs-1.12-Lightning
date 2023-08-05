<template>
  <canvas ref="canvasRef"></canvas>
</template>

<script setup>
import { ref, onMounted, onUnmounted, guardReactiveProps} from "vue"
import { OrbitControls} from "three/examples/jsm/controls/OrbitControls"
import { RectAreaLightHelper } from "three/examples/jsm/helpers/RectAreaLightHelper"
import * as THREE from "three"

var controls


let canvasRef = ref();

//Utilizziamo i valori del viewport per impostare la grandezza del canvas
const sizes = {
  width: window.innerWidth,
  height: window.innerHeight
}

//Aggiungiamo l'event listner per il resize del canvas
window.addEventListener("resize", ()=>{
  //update sizes for the canvas
  sizes.width = window.innerWidth
  sizes.height = window.innerHeight

  //update the camera aspect ratio
  camera.aspect = sizes.width / sizes.height
  camera.updateProjectionMatrix()

  //update renderer
  renderer.setSize(sizes.width, sizes.height)
  //HANDLE PIXEL RATIO
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
})


//HANDLE FULL SCREEN (Il codice aggiuntivo è per permettere il funzionamento corretto anche con browser Safari)
window.addEventListener("dblclick",()=>{

  const fullscreenElement = document.fullscreenElement ||  document.webkitFullscreenElement
  if(!fullscreenElement){
    if(canvasRef.value.requestFullscreen){
      canvasRef.value.requestFullscreen()
    }
    else if(canvasRef.value.webkitRequestFullscreen){
      canvasRef.value.webkitRequestFullscreen()
    }
    
  }
  else{
    if(document.exitFullscreen)
    {
      document.exitFullscreen()
    }
    else if(document.webkitExitFullscreen){
      document.webkitExitFullscreen()
    }
  }
})

let scene = new THREE.Scene()

let renderer

//LIGHTS
//Adding light is simple, just instantiate the right class of light and add it to the scene
// You can change the light positions and rotate it and many other things.
//you can aso use lookAt(objectToLook or Vector3 Position), to rotate more easly 

//NOTE that light can cost a lot in terms of performances, so try add as few lights as possible and try to use the light that cost less like:
// minimal cost: ambientLight, HemisphereLight
// medium cost: directionalLight, pointLight
// high cost: spotLight, rectAreaLight




//Ambient Light 

//This is a light that will illuminate all the scene ambient, indipendently from the position (is an omnidirectional light)
//We can use this light also for simulate the light bouncing that give us the possibility of look at the non directly illuminated part of an object ( the light buoncing is hard to calculate in real time, so we need to simulate it in a much simpler way like this)

const ambientLight = new THREE.AmbientLight(0xffffff, 0.5) //The first parameter is the color of the light, the second parameter is the intensity of the light

//We can also change the color and the intensity even in a second moment , doing this:
ambientLight.color = new THREE.Color(0xffffff)
ambientLight.intensity = 0.5

scene.add(ambientLight)




//Directional Light



//The directional light imitate the sun, the rays of light go in parallel giving this effect whre we can see the light mush on the part directly illuminated

const directionalLight = new THREE.DirectionalLight(0x00fffc, 0.3)
scene.add(directionalLight)

// you can change the orientation of the light, using it position
directionalLight.position.set(1,0.25,0)






//Hemisphere Light
//Similar to the ambient light but with 2 different color: 1 for the light coming from the sky, 1 for the light coming from the ground
const hemisphereLight = new THREE.HemisphereLight(0xff0000, 0x0000ff, 0.3) //first parameter sky light, the second parameter groud light, third parameter intensity
scene.add(hemisphereLight)





//Point Light
//this one is like a lighter, is starts at an infinitely small point and spread uniformly in every direction

//the pointlight have other parameters: distance and decay
//distance is the fade distance, and the decay is how fast it fades

const pointLight = new THREE.PointLight(0xff9000, 0.5,10,2)
pointLight.position.set(1,-0.5,1)
scene.add(pointLight)





//Rect Area Light 

//NOTE that this light works only with MeshStandardMaterial and MeshPhysicalMaterial

//the rect area light is like the big light you can see on a photoshoot set
//it's a mix between a directional light and a diffuse light
const rectAreaLight = new THREE.RectAreaLight(0x4e00ff, 2, 1, 1) //color, intensity, width, height
rectAreaLight.position.set(-1.5, 0, 1.5)
rectAreaLight.lookAt(new THREE.Vector3())
scene.add(rectAreaLight)




//Spot Light

//the spotlight is like an imitation of a flashlight (una torcia), that illuminate a specific area, and its "end" is a cirle

//the parameters are : color, intensity, distance (max at which the light illuminate), angle (how wide is you light angle) , penumbra (how much of the object not illuminated by the light you can see, and how much it blur on the side), decay (how fast it fades , usually just keep 1)
const spotLight = new THREE.SpotLight(0x78ff00, 0.5, 10, Math.PI* 0.1, 0.25, 0.1) //Math.PI * 0.1 = 180 degrees * 0.1 , or 180/10 so 18 degrees, and is the wide of the angle of the light source
spotLight.position.set(0, 2, 3)
scene.add(spotLight)

//For looking at something with the spotlight, it differ from the other light, we should use the target instead of lookAt


//the target is a teoretical object that we can't see, but 
//but first we need to add this spotlight target to the scene doing this:
scene.add(spotLight.target)
spotLight.target.position.x = -1.75



//BAKING LIGHTS 
// we can't use so many light in our scene, so when we need a more detailed scene we can avoid this problem using a tecnique called baking
//baking consist of bake the light into the texture in a 3D software during the object creation
//the drawback is that we cannot move the light anymore and we have to load huge textures


//LIGHT HELPER
//positioning the  lights can be very hard, because we can't see it's position. 
//to solve this problem we can use the light helpers
//there are many types of helper depending on the light that you use
// DirectionalLightHelper, RectAreaHelper, SpotLightHelper, PointLightHelper, HemisphereLightHelper

/*

const hemisphereLightHelper = new THREE.HemisphereLightHelper(hemisphereLight, 0.2) //the first parameters is the light for we want helper, the second parameters is the size of the helper
scene.add(hemisphereLightHelper)

const directionalLightHelper = new THREE.DirectionalLightHelper(directionalLight, 0.2)
scene.add(directionalLightHelper)

const pointLightHelper = new THREE.PointLightHelper(pointLight, 0.2)
scene.add(pointLightHelper)

const spotLightHelper = new THREE.SpotLightHelper(spotLight)
scene.add(spotLightHelper)

//for the rectArea helper it's a little different, we need to import the helper in the first section of the script, or it doesn't work
const rectAreaHelper = new RectAreaLightHelper(rectAreaLight) 
scene.add(rectAreaHelper)
*/



// BOX
const boxGeometry = new THREE.BoxGeometry(1,1,1)
const boxMaterial = new THREE.MeshStandardMaterial() 
boxMaterial.roughness = 0.4
const box = new THREE.Mesh(boxGeometry,boxMaterial)


box.position.set(0 , 0, 0)

scene.add(box)

// SPHERE
const sphereGeometry = new THREE.SphereBufferGeometry(0.5,32,32)
const sphereMaterial = new THREE.MeshStandardMaterial() 
sphereMaterial.roughness = 0.4
const sphere = new THREE.Mesh(sphereGeometry,sphereMaterial)


sphere.position.set(-3 , 0, 0)

scene.add(sphere)

// TORUS
const torusGeometry = new THREE.TorusBufferGeometry()
const torusMaterial = new THREE.MeshStandardMaterial() 
torusMaterial.roughness = 0.4
const torus = new THREE.Mesh(torusGeometry,torusMaterial)


torus.position.set(3 , 0, 0)

scene.add(torus)

// PLANE
const planeGeometry = new THREE.PlaneBufferGeometry(10,10)
const planeMaterial = new THREE.MeshStandardMaterial() 
planeMaterial.roughness = 0.4
const plane = new THREE.Mesh(planeGeometry,planeMaterial)


plane.position.set(0 , -2 , 0)
plane.rotation.x -=  Math.PI /2

scene.add(plane)

// CAMERA
let camera = new THREE.PerspectiveCamera(
  75, 
  sizes.width / sizes.height, 
  0.1,
  100  
);

camera.position.set(0,0,3)
scene.add(camera);

const clock = new THREE.Clock()

const animate = () =>{
  const elapsedTime = clock.getElapsedTime()

  controls.update()
  
  renderer.render(scene, camera)
}

onMounted(() => {
  renderer = new THREE.WebGLRenderer({
    canvas: canvasRef.value
  });
 
  renderer.setSize(sizes.width, sizes.height)
  //HANDLE PIXEL RATIO
  //è possivile che veriamo un po di imperfezioni sul render, soprattutto sugli angoli
  //il pixel ratio corrisponde a quanti pixel fisici hai sul tuo schermo, per un unita di pixel del software
  renderer.setPixelRatio(Math.min(window.devicePixelRatio,2)) // in questo modo il max valore di pixel ratio utilizzaro è 2 (altrimenti devi renderizzare troppi pixel, dispiego enorme di potenza gpu)
  renderer.render(scene, camera)
  renderer.setAnimationLoop(animate)

  

  //CONTROLS
  controls = new OrbitControls(camera, canvasRef.value)
  controls.enableDamping = true; // permette uno effetto smooth una volta che rilasciamo l'elemento, rallenta fino a fermarsi
});

onUnmounted(() => {
  renderer.setAnimationLoop(null)
});

</script>

<style>
canvas {
  width: 100%;
  height: 100%;
  display: block;
}
</style>
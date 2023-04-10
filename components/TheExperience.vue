<template>
    <canvas ref="canvas" class="canvas"></canvas>
</template>


<script setup lang="ts">
import {
    Scene, BoxGeometry, Color, GridHelper, DirectionalLight, PerspectiveCamera, Mesh,
    SphereGeometry, MeshBasicMaterial, WebGLRenderer, AmbientLight, Clock,
    MeshStandardMaterial, DirectionalLightHelper, HemisphereLight, HemisphereLightHelper,
    PointLight, PointLightHelper, MeshNormalMaterial
} from 'three';
import { OBJLoader } from "three/examples/jsm/loaders/OBJLoader";
import { GUI } from 'three/examples/jsm/libs/lil-gui.module.min.js';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { ref, onMounted } from 'vue';
import { timestamp, useWindowSize } from '@vueuse/core';

//debug
const gui = new GUI();

//scene
const scene = new Scene();


//Light
//1) AmnientLight
const ambientLight = new AmbientLight('#ffffff', 0.5);
scene.add(ambientLight);
gui.add(ambientLight, "intensity")
    .min(0)
    .max(1)
    .step(0.1)
    .name("環境光");
//2) DirectionalLight
const directionalLight = new DirectionalLight("#ffffff", 0.5);
scene.add(directionalLight);
directionalLight.position.set(0, 2, 0);
gui.add(directionalLight, "intensity")
    .min(0)
    .max(1)
    .step(0.1)
    .name("定向光");
gui.add(directionalLight.position, "x").min(-3).max(3).step(0.01).name("directionalLight X Position");
gui.add(directionalLight.position, "y").min(-3).max(3).step(0.01).name("directionalLight Y Position");
//DirectionalLightHelper
const directionalLightHelper = new DirectionalLightHelper(directionalLight);
scene.add(directionalLightHelper);
//3)HemisphereLight
const hemisphereLight = new HemisphereLight("blue", "yellow", 0.5);
scene.add(hemisphereLight);
gui.add(hemisphereLight, "intensity").min(0).max(1).step(0.1).name("hemisphereLight intensity");
//HemisphereLightHelper
// const hemisphereLightHelper = new HemisphereLightHelper(hemisphereLight, 5);
// scene.add(hemisphereLightHelper);

//4) Point Light
const pointLight = new PointLight("red", 1.5, 5)
scene.add(pointLight);
gui.add(pointLight.position, "x").min(-3).max(3).step(0.01).name("pointLight X ");
gui.add(pointLight.position, "y").min(-3).max(3).step(0.01).name("pointLight Y ");
gui.add(pointLight.position, "z").min(-3).max(3).step(0.01).name("pointLight Z ");
//Point LightHelper
const pointLightHelper = new PointLightHelper(pointLight);
scene.add(pointLightHelper);

//

//Mesh
const geometry = new BoxGeometry(1, 1, 1);
const material = new MeshStandardMaterial({ color: "white", wireframe: false });
const mesh = new Mesh(geometry, material);
scene.add(mesh);

//camera
const aspect = {
    width: window.innerWidth,
    height: window.innerHeight,
};

const camera = new PerspectiveCamera(75, aspect.width / aspect.height);

scene.add(camera);
camera.position.x = 2;
camera.position.y = 2;
camera.position.z = 2;
camera.lookAt(mesh.position);

//canvas
const canvas: Ref<HTMLCanvasElement | null> = ref(null);
let orbitControls: OrbitControls;
let renderer: WebGLRenderer;


//Clock class 讓每台設備fps問題 移動量問題一置
const clock = new Clock();
let elapsedTime: number;



//model
const objloader = new OBJLoader();
objloader.load('/models/suzan.obj', (object) => {
    console.log(object);
    object.position.y = 1;
    object.children[0].position.z = -3;
    (object.children[0] as Mesh).material = new MeshNormalMaterial();
    scene.add(object);
});




const animate = () => {
    //GetElapsedTome
    elapsedTime = clock.getElapsedTime();
    // console.log("秒數 :", elapsedTime);
    mesh.rotation.y = Math.PI * elapsedTime * 0.1;
    //---------------------------------Update OrbitControls------------------------------------
    orbitControls.update();
    //--------------------------------------Renderer-------------------------------------------
    renderer.render(scene, camera);
    //---------------------------------RequestAnimationFrame-----------------------------------
    window.requestAnimationFrame(animate);
};


onMounted(() => {
    renderer = new WebGLRenderer({ canvas: canvas.value! });
    renderer.setSize(aspect.width, aspect.height);
    renderer.render(scene, camera);
    orbitControls = new OrbitControls(camera, canvas.value!);
    animate();
});

</script>

<style>
.canvas {
    position: fixed;
    margin: 0;
    padding: 0;
}
</style>
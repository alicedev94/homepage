<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { OBJLoader } from 'three/examples/jsm/loaders/OBJLoader';
import { MTLLoader } from 'three/examples/jsm/loaders/MTLLoader';
import Aincrad from '@/assets/aincrad/Aincrad.obj';
import Textures from '@/assets/aincrad/Aincrad.mtl';

const threeContainer = ref(null);

onMounted(() => {
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
    const renderer = new THREE.WebGLRenderer({ alpha: true });

    if (threeContainer.value) {
        const { clientWidth: width, clientHeight: height } = threeContainer.value;
        renderer.setSize(width, height);
        camera.aspect = width / height;
        camera.updateProjectionMatrix();
        threeContainer.value.appendChild(renderer.domElement);
    }

    const controls = new OrbitControls(camera, renderer.domElement);
    controls.enablePan = false; // Desactivar panorámico
    controls.enableRotate = false; // Desactivar rotación
    controls.enableDamping = true; // Añadir inercia
    controls.dampingFactor = 0.25;
    controls.maxDistance = 50; // Máxima distancia de zoom
    controls.minDistance = 5; // Mínima distancia de zoom

    // Permitir scroll cuando el cursor esté sobre el modelo
    const allowScroll = (event) => {
        event.stopPropagation();
    };

    threeContainer.value.addEventListener('wheel', allowScroll);

    const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
    scene.add(ambientLight);

    const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
    directionalLight.position.set(5, 10, 7.5);
    scene.add(directionalLight);

    const mtlLoader = new MTLLoader();
    let model;

    mtlLoader.load(Textures, (materials) => {
        materials.preload();
        const objLoader = new OBJLoader();
        objLoader.setMaterials(materials);
        objLoader.load(Aincrad, (object) => {
            model = object;
            scene.add(model);
            model.position.set(0, -2, -450);
        }, undefined, (error) => {
            console.error('Error al cargar el modelo:', error);
        });
    });

    camera.position.z = 5;

    function resizeRendererToDisplaySize() {
        if (threeContainer.value) {
            const width = threeContainer.value.clientWidth;
            const height = threeContainer.value.clientHeight;
            const needResize = renderer.domElement.width !== width || renderer.domElement.height !== height;
            if (needResize) {
                renderer.setSize(width, height, false);
                camera.aspect = width / height;
                camera.updateProjectionMatrix();
            }
        }
    }

    function animate() {
        resizeRendererToDisplaySize();
        requestAnimationFrame(animate);
        if (model) {
            model.rotation.y += 0.01;
        }
        controls.update();
        renderer.render(scene, camera);
    }

    animate();

    onUnmounted(() => {
        renderer.dispose();
        if (threeContainer.value) {
            threeContainer.value.removeEventListener('wheel', allowScroll);
            threeContainer.value.removeChild(renderer.domElement);
        }
    });
});

</script>

<template>
    <div class="content-model">
        <div class="model" ref="threeContainer"></div>
    </div>
</template>

<style scoped>
.content-model {
    display: flex;
    justify-content: center;
    height: 30rem;
    width: 100%;
    /* background-color: aquamarine; */
}

.model {
    display: flex;
    justify-content: center;
    align-content: center;
    height: 30rem;
    width: 50rem;
    position: relative;
    overflow: hidden;
}
</style>

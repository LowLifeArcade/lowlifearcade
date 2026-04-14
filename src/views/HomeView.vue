<template>
    <div class="globe-wrapper">
        <div class="ambient-glow" />
        <canvas ref="canvasRef" class="globe-canvas" />
        <div class="label">
            <span class="dot" />
            SCANNING
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, render } from 'vue';
import * as THREE from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader';
import { OrbitControls } from 'three/addons/controls/OrbitControls';
import { EffectComposer } from 'three/addons/postprocessing/EffectComposer';
import { RenderPass } from 'three/addons/postprocessing/RenderPass';
import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass';

const canvasRef = ref(null);
const loader = new GLTFLoader();

let animationId = null;
let renderer = new THREE.WebGLRenderer(),
    scene = new THREE.Scene(),
    camera = new THREE.PerspectiveCamera(),
    globe = new THREE.Mesh(),
    stars = new THREE.Points(),
    spaceship,
    composer = new EffectComposer(renderer),
    orbitAngle = 2;
const orbitRadius = 2.5,
    orbitSpeed = 0.03;

function init() {
    const canvas = canvasRef.value;
    const W = canvas.clientWidth;
    const H = canvas.clientHeight;

    // Renderer
    renderer = new THREE.WebGLRenderer({ canvas, antialias: true, alpha: true });
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setSize(W, H);

    // Scene & Camera
    scene = new THREE.Scene();
    camera = new THREE.PerspectiveCamera(45, W / H, 0.1, 100);
    camera.position.z = 5.8;

    const listener = new THREE.AudioListener();
    camera.add(listener);

    // 2. Create audio object
    const sound = new THREE.Audio(listener);

    // 3. Load audio file
    const audioLoader = new THREE.AudioLoader();
    audioLoader.load('/sounds/spaceloop.mp3', function (buffer) {
        sound.setBuffer(buffer);
        sound.setLoop(true);
        sound.setVolume(0.5);
        sound.play();
    });

    // 🔑 unlock audio on first user interaction
    function unlockAudio() {
        const context = listener.context;
        console.log({ context })

        if (context.state === 'suspended') {
            context.resume();
        }

        document.removeEventListener('pointerdown', unlockAudio);
    }

    document.addEventListener('pointerdown', unlockAudio);

    // ── Globe ──────────────────────────────────────────────
    const geo = new THREE.SphereGeometry(1, 64, 64);

    // Wireframe overlay mesh (latlon lines)
    // const wireMat = new THREE.MeshBasicMaterial({
    //     color: 0x00e5ff,
    //     wireframe: true,
    //     transparent: true,
    //     opacity: 0.08,
    // });
    // const wireGlobe = new THREE.Mesh(geo, wireMat);
    // scene.add(wireGlobe);

    // Solid base sphere
    const mat = new THREE.MeshPhongMaterial({
        // color: 0x0a1a2e,
        // emissive: 0x0d2d50,
        // emissiveIntensity: 0.4,
        // shininess: 60,
        // transparent: true,
        // opacity: 0.95,
    });

    const textureURL = './moon.jpg';
    const displacementURL = './moon-2.jpg';
    // const textureURL = 'https://s3-us-west-2.amazonaws.com/s.cdpn.io/17271/lroc_color_poles_1k.jpg';
    // const displacementURL = 'https://s3-us-west-2.amazonaws.com/s.cdpn.io/17271/ldem_3_8bit.jpg';
    // const worldURL = 'https://s3-us-west-2.amazonaws.com/s.cdpn.io/17271/hipp8_s.jpg';

    // const scene = new THREE.Scene();

    // const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);

    // const renderer = new THREE.WebGLRenderer();

    const controls = new OrbitControls(camera, renderer.domElement);
    // controls.enablePan = false;

    // renderer.setSize(window.innerWidth, window.innerHeight);
    // document.body.appendChild(renderer.domElement);

    // const geometry = new THREE.SphereGeometry(2, 60, 60);

    const textureLoader = new THREE.TextureLoader();
    const texture = textureLoader.load(textureURL);
    const displacementMap = textureLoader.load(displacementURL);

    const material = new THREE.MeshPhongMaterial({
        color: 0xffffff,
        map: texture,
        displacementMap: displacementMap,
        displacementScale: 0,
        bumpMap: displacementMap,
        // bumpScale: 0.01,
        reflectivity: 0,
        shininess: 0,
    });

    globe = new THREE.Mesh(geo, material);
    scene.add(globe);

    // -- SUN --
    const bigSunGeo = new THREE.SphereGeometry(1.2, 64, 64);
    const sunMesh = new THREE.MeshBasicMaterial({});
    const bigSun = new THREE.Mesh(bigSunGeo, sunMesh);
    bigSun.position.z = -80.18;
    scene.add(bigSun);
    // -- Earch --
    const earthGeo = new THREE.SphereGeometry(3, 64, 64);
    const earthMesh = new THREE.MeshPhongMaterial({
        color: 0xaaafff,
        emissive: 1,
        transparent: false,
    });
    const earth = new THREE.Mesh(earthGeo, earthMesh);
    earth.position.x = -50.18;
    // earth.position.z = 20.18;
    // earth.position.y = 10
    scene.add(earth);

    composer = new EffectComposer(renderer);
    composer.addPass(new RenderPass(scene, camera));

    const bloomPass = new UnrealBloomPass(
        new THREE.Vector2(window.innerWidth, window.innerHeight),
        1.5, // strength
        0.4, // radius
        0.002, // threshold
    );
    composer.addPass(bloomPass);

    // ── Atmosphere glow (slightly larger sphere) ───────────
    // const atmGeo = new THREE.SphereGeometry(1.12, 64, 64);
    // const atmMat = new THREE.MeshPhongMaterial({
    //     color: 0xdff0ff,
    //     transparent: true,
    //     opacity: 0.06,
    //     side: THREE.FrontSide,
    // });
    // const atmosphere = new THREE.Mesh(atmGeo, atmMat);
    // scene.add(atmosphere);

    // ── Stars ──────────────────────────────────────────────
    const starGeo = new THREE.BufferGeometry();
    const starCount = 3800;
    const positions = new Float32Array(starCount * 3);
    for (let i = 0; i < starCount * 3; i++) {
        positions[i] = (Math.random() - 0.5) * 100;
    }

    starGeo.setAttribute('position', new THREE.BufferAttribute(positions, 3));
    stars = new THREE.Points(
        starGeo,
        new THREE.PointsMaterial({
            color: 0xffffff,
            size: 0.06,
            transparent: true,
            opacity: 0.7,
        }),
    );
    scene.add(stars);

    // ── Lights ─────────────────────────────────────────────
    // scene.add(new THREE.AmbientLight(0x1a3a6e, 1.2));

    const sun = new THREE.DirectionalLight(0xffffff, 0.41);
    sun.position.set(6.5, 1, -16);
    scene.add(sun);

    // use this and remove sun for similar eclipse image (not original we have here)
    // const rim = new THREE.DirectionalLight('88ccff', 0.06);
    const rim = new THREE.DirectionalLight(0xd2dfff, 0.51);
    rim.position.set(-4, 1, -2);
    scene.add(rim);

    // ── Resize ─────────────────────────────────────────────
    window.addEventListener('resize', onResize);

    loader.load(
        './spaceship_eav_2_crab.glb',
        function (gltf) {
            spaceship = gltf.scene;
            scene.add(spaceship);
            spaceship.traverse((node) => (node.castShadow = true));

            spaceship.scale.set(0.01, 0.01, 0.01);
            spaceship.position.set(orbitRadius, 0.2, 0);
        },
        function (xhr) {
            console.log((xhr.loaded / xhr.total) * 100 + '% loaded');
        },
        function (error) {
            console.error('An error occurred loading the GLB:', error);
        },
    );
}

function onResize() {
    const canvas = canvasRef.value;
    if (!canvas) return;
    const W = canvas.clientWidth;
    const H = canvas.clientHeight;
    camera.aspect = W / H;
    camera.updateProjectionMatrix();
    renderer.setSize(W, H);
}

function animate() {
    animationId = requestAnimationFrame(animate);

    if (spaceship) {
        // Update orbit angle
        orbitAngle += orbitSpeed * 10;

        // Calculate new position
        const radians = orbitAngle * (Math.PI / 180);
        spaceship.position.x = Math.cos(radians) * orbitRadius;
        spaceship.position.z = Math.sin(radians) * orbitRadius;

        // Make spaceship face direction of travel
        spaceship.rotation.y = -radians + Math.PI / 2;
    }

    globe.rotation.y += 0.0025;
    stars.rotation.y += 0.0002;
    composer.render(scene, camera);
}

onMounted(() => {
    init();
    animate();
});

onBeforeUnmount(() => {
    cancelAnimationFrame(animationId);
    window.removeEventListener('resize', onResize);
    renderer.dispose();
});
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Share+Tech+Mono&display=swap');

.globe-wrapper {
    position: relative;
    width: 100%;
    height: 100vh;
    background: radial-gradient(#273039 10%, #000307 55%);
    overflow: hidden;
    display: flex;
    align-items: center;
    justify-content: center;
}

/* soft cyan bloom behind the globe */
.ambient-glow {
    position: absolute;
    width: 520px;
    height: 520px;
    border-radius: 50%;
    background: radial-gradient(circle, rgba(28, 32, 34, 0.12) 0%, transparent 70%);
    pointer-events: none;
}

.globe-canvas {
    width: 100%;
    height: 100%;
    display: block;
}

/* HUD label bottom-left */
.label {
    position: absolute;
    bottom: 2rem;
    left: 2rem;
    font-family: 'Share Tech Mono', monospace;
    font-size: 0.7rem;
    letter-spacing: 0.25em;
    color: rgba(0, 200, 255, 0.5);
    display: flex;
    align-items: center;
    gap: 0.5rem;
    user-select: none;
}

.dot {
    width: 6px;
    height: 6px;
    border-radius: 50%;
    background: #00e5ff;
    box-shadow: 0 0 6px #00e5ff;
    animation: pulse 2s ease-in-out infinite;
}

@keyframes pulse {
    0%,
    100% {
        opacity: 1;
        transform: scale(1);
    }
    50% {
        opacity: 0.4;
        transform: scale(0.7);
    }
}
</style>

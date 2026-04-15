<template>
    <div class="globe-wrapper">
        <div class="ambient-glow" />
        <canvas
            ref="canvasRef"
            class="globe-canvas"
        />
        <div
            v-if="state.game"
            class="label"
        >
            <span class="dot" />
            {{ state.game.toUpperCase() }}
            <span v-if="loadingBar !== 100">% {{ loadingBar }}</span>
        </div>
        <div class="middle">Nav: use arrow keys</div>
        <div class="hud">
            <ul>
                <li>{{ state.scene }} :scene</li>
                <li class="music">
                    <div @click="onToggleSound">{{ state.audio === AL_STATE.running ? 'on' : 'off' }} :sound</div>
                    <div class="credits">
                        music by
                        <a
                            href="https://pixabay.com/users/idoberg-34953295/"
                            target="_blank"
                            >IdoBerg</a
                        >
                    </div>
                </li>
            </ul>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, render, computed, watch, reactive } from 'vue';
import * as THREE from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader';
import { OrbitControls } from 'three/addons/controls/OrbitControls';
import { EffectComposer } from 'three/addons/postprocessing/EffectComposer';
import { RenderPass } from 'three/addons/postprocessing/RenderPass';
import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass';

const NAV_KEYS = {
    forward: ['Enter', 'ArrowRight', 'ArrowUp', ' '],
    backward: ['ArrowLeft', 'ArrowDown'],
};
const SCENES = {
    sun: 'sun',
    earth: 'earth',
    moon: 'moon',
    space: 'space',
};
const AL_STATE = {
    running: 'running',
    suspended: 'suspended',
};
const GAME_STATE = {
    scanning: 'scanning',
    loading: 'loading',
};

const canvasRef = ref(null);

const state = reactive({
    scene: SCENES.moon,
    game: GAME_STATE.loading,
    audio: AL_STATE.running,
});
const loader = new GLTFLoader();
const loadingBar = ref(0);
const ctrl = new AbortController();
const eventOpts = { signal: ctrl.signal };

let animationId = null;
let renderer = new THREE.WebGLRenderer(),
    scene = new THREE.Scene(),
    camera = new THREE.PerspectiveCamera(),
    globe = new THREE.Mesh(),
    stars = new THREE.Points(),
    composer = new EffectComposer(renderer),
    audioListener = new THREE.AudioListener(),
    sound = new THREE.Audio(audioListener),
    spaceship,
    earth,
    moving = true,
    switchable = true,
    controls,
    orbitAngle = 2,
    orbitRadius = 2.5,
    orbitSpeed = 0.03;

function onToggleSound() {
    if (audioListener?.context?.state === 'suspended') {
        audioListener?.context.resume();
        state.audio = AL_STATE.running;
    } else {
        audioListener?.context.suspend();
        state.audio = AL_STATE.suspended;
    }
}

function toggleState(key) {
    if (!switchable) {
        return;
    }

    controls.enabled = false;

    if (state.scene === SCENES.moon) {
        if (NAV_KEYS.forward.includes(key)) {
            state.scene = SCENES.sun;
        } else {
            state.scene = SCENES.earth;
        }

        moving = true;
        switchable = false;
    } else if (state.scene === SCENES.sun) {
        if (NAV_KEYS.forward.includes(key)) {
            state.scene = SCENES.earth;
        } else {
            state.scene = SCENES.moon;
        }

        moving = true;
        switchable = false;
    } else if (state.scene === SCENES.earth) {
        if (NAV_KEYS.forward.includes(key)) {
            state.scene = SCENES.moon;
        } else {
            state.scene = SCENES.sun;
        }

        moving = true;
        switchable = false;
    }
}

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

    camera.add(audioListener);

    const audioLoader = new THREE.AudioLoader();
    audioLoader.load('/sounds/spaceloop.mp3', function (buffer) {
        sound.setBuffer(buffer);
        sound.setLoop(true);
        sound.setVolume(0.5);
        sound.play();
    });

    function unlockAudio(e) {
        const keys = ['Enter', 'ArrowRight', 'ArrowLeft', 'ArrowUp', 'ArrowDown', ' '];
        if (e?.key && !keys.includes(e?.key)) {
            return;
        }

        if (audioListener?.context?.state === 'suspended') {
            audioListener?.context.resume();
        }

        document.removeEventListener('pointerdown', unlockAudio);
        document.removeEventListener('keydown', unlockAudio);
    }

    document.addEventListener('pointerdown', unlockAudio, eventOpts);
    document.addEventListener('keydown', unlockAudio, eventOpts);
    document.addEventListener(
        'keydown',
        (e) => {
            const keys = ['Enter', 'ArrowRight', 'ArrowLeft', 'ArrowUp', 'ArrowDown', ' '];
            if (keys.includes(e.key)) {
                toggleState(e.key);
            }
        },
        eventOpts,
    );

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

    controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;
    controls.dampingFactor = 0.08;

    // controls.addEventListener('change', () => {
    //     console.log(camera.position);
    // });

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
        // emissive: 1,
        // transparent: false,
        emissive: 0x000333,
        emissiveIntensity: 0.02,
        // shininess: 20,
        // transparent: true,
        // opacity: 0.95,
    });

    earth = new THREE.Mesh(earthGeo, earthMesh);
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
    const sun = new THREE.DirectionalLight(0xffffff, 0.41);
    sun.position.set(6.5, 1, -16);
    scene.add(sun);

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
            loadingBar.value = (xhr.loaded / xhr.total) * 100;

            if ((xhr.loaded / xhr.total) * 100 === 100) {
                state.game = GAME_STATE.scanning;
            }
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

const currentLookAt = new THREE.Vector3();
const targets = {
    sun: {
        camera: new THREE.Vector3(1.614101934079007, 0.05827171062183323, 1.63277437998612),
        loc: new THREE.Vector3(1.2, 0, 0),
    },
    earth: {
        camera: new THREE.Vector3(1.187330832077383, 1.3996101001244262, 1.9398882467499785),
        loc: new THREE.Vector3(-1.2, 1, 1),
    },
    moon: {
        camera: new THREE.Vector3(0.02705811711985545, 0.20869967182489138, 4.284386682929132),
        loc: new THREE.Vector3(0, 0, 0),
    },
};

function handleSceneSwitch(target = { loc: '', camera: '' }) {
    if (!moving) {
        return;
    }

    camera.position.lerp(target.camera, 0.05); // smooth factor (0–1)
    currentLookAt.lerp(target.loc, 0.05);
    camera.lookAt(currentLookAt);

    const positionSwitchable = camera.position.distanceTo(target.camera) < 0.3;
    const lookSwitchable = currentLookAt.distanceTo(target.loc) < 0.3;
    const positionDone = camera.position.distanceTo(target.camera) < 0.001;
    const lookDone = currentLookAt.distanceTo(target.loc) < 0.001;

    if (positionSwitchable && lookSwitchable) {
        switchable = true;
    }

    if (positionDone && lookDone) {
        camera.position.copy(target.camera);
        controls.target.copy(target.loc);
        controls.update();
        moving = false;
    }
}

function animate() {
    animationId = requestAnimationFrame(animate);
    controls.update();

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

    handleSceneSwitch(targets[state.scene]);

    if (moving) {
        controls.enabled = false;
    } else {
        controls.enabled = true;
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

    if (sound) {
        sound.stop();
        sound.disconnect();
    }

    if (audioListener) {
        camera.remove(audioListener);
        audioListener.context.close();
    }

    ctrl.abort();
    renderer.dispose();
});
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Share+Tech+Mono&display=swap');

.globe-wrapper {
    font-family: 'Share Tech Mono', monospace;
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

.label {
    position: absolute;
    bottom: 2rem;
    left: 2rem;
    font-size: 0.7rem;
    letter-spacing: 0.25em;
    color: rgba(0, 200, 255, 0.5);
    display: flex;
    align-items: center;
    gap: 0.5rem;
    user-select: none;

    .dot {
        width: 6px;
        height: 6px;
        border-radius: 50%;
        background: #00e5ff;
        box-shadow: 0 0 6px #00e5ff;
        animation: pulse 2s ease-in-out infinite;
    }
}

.hud {
    font-size: 1.2rem;
    position: absolute;
    bottom: 2rem;
    right: 2rem;
    color: rgb(106, 168, 197);

    ul {
        list-style: none;

        li {
            cursor: pointer;
            text-align: end;
        }
    }

    .music {
        padding: unset;

        .credits {
            font-size: 0.8rem;
        }
    }
}

.middle {
    position: absolute;
    bottom: 2rem;
    color: white;
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

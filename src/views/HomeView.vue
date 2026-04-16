<template>
    <div class="globe-wrapper">
        <div class="ambient-glow" />
        <canvas
            ref="canvasRef"
            class="globe-canvas"
        />

        <div class="label">
            <ul>
                <li
                    v-for="(msg, i) in displayMsgs"
                    class="msg"
                >
                    <span>
                        {{ msg.toUpperCase() }}
                    </span>
                    <span
                        v-if="i + 1 === displayMsgs.length"
                        class="dot"
                    />
                </li>
            </ul>
        </div>
        <div class="middle">
            <span v-if="hudActive">Nav: use arrow keys</span>
        </div>
        <div class="onscreen">
            <span v-if="!hudActive">Press Enter</span>
        </div>
        <div
            v-if="hudActive"
            class="hud"
        >
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
// .js is needed for type inference
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass.js';
import { MSGS } from '../consts/msgs'

const NAV_KEYS = {
    forward: ['Enter', 'ArrowRight', 'ArrowUp', ' '],
    backward: ['ArrowLeft', 'ArrowDown'],
    exit: ['Escape'],
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
    active: 'active',
    loading: 'loading',
};

const canvasRef = ref(null);

const state = reactive({
    scene: SCENES.space,
    game: GAME_STATE.loading,
    audio: AL_STATE.running,
});
const switchable = ref(false);
const game = reactive({
    controls: null,
});
const loader = new GLTFLoader();
const loadingBar = ref(0);
const ctrl = new AbortController();
const eventOpts = { signal: ctrl.signal };
const hudActive = computed(() => state.scene !== SCENES.space);

let id;
const displayMsgs = ref([]);

let int;
watch(
    () => state.scene,
    async () => {
        displayMsgs.value = [];
        if (int) {
            clearInterval(int);
        }
        if (id) {
            clearTimeout(id);
        }

        for await (const [msg, time, wordSpeed] of MSGS[state.scene]) {
            displayMsgs.value.push('');

            await new Promise((res) => (id = setTimeout(res, time)));
            await new Promise((res) => {
                let i = 0;
                let curr = '';

                int = setInterval(() => {
                    curr = msg[i];
                    i++;

                    displayMsgs.value[displayMsgs.value?.length - 1] += curr;

                    if (i >= msg.length) {
                        clearInterval(int);
                        res();
                    }
                }, wordSpeed || 30);
            });
        }
    },
    { immediate: true },
);

let animationId = null;
let renderer = new THREE.WebGLRenderer(),
    scene = new THREE.Scene(),
    camera = new THREE.PerspectiveCamera(),
    globe = new THREE.Mesh(),
    stars = new THREE.Points(),
    composer = new EffectComposer(renderer),
    audioListener = new THREE.AudioListener(),
    sound = new THREE.Audio(audioListener),
    /** @type {THREE.Group<THREE.Object3DEventMap>} */
    spaceship,
    earth,
    moving = true,
    orbitAngle = 2,
    orbitRadius = 2.5,
    orbitSpeed = 0.03;

watch(
    switchable,
    (s) => {
        if (game.controls) {
            game.controls.enabled = s;
            game.controls.update();
        }
    },
    { immediate: true, deep: true },
);

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
    if (NAV_KEYS.exit.includes(key) && audioListener?.context?.state === 'running') {
        state.scene = SCENES.space;
        audioListener?.context.suspend();
        moving = true;
        switchable.value = false;
        return;
    }

    if (!game.controls.enabled && key !== 'Enter') {
        return;
    }

    if (state.audio === AL_STATE.running && audioListener?.context?.state === 'suspended') {
        audioListener?.context?.resume();
    }

    if (state.scene === SCENES.moon) {
        if (NAV_KEYS.forward.includes(key)) {
            state.scene = SCENES.sun;
        } else {
            state.scene = SCENES.earth;
        }

        moving = true;
        switchable.value = false;
    } else if (state.scene === SCENES.sun) {
        if (NAV_KEYS.forward.includes(key)) {
            state.scene = SCENES.earth;
        } else {
            state.scene = SCENES.moon;
        }

        moving = true;
        switchable.value = false;
    } else if (state.scene === SCENES.earth) {
        if (NAV_KEYS.forward.includes(key)) {
            state.scene = SCENES.moon;
        } else {
            state.scene = SCENES.sun;
        }

        moving = true;
        switchable.value = false;
    } else if (state.scene === SCENES.space) {
        state.scene = SCENES.moon;
        moving = true;
        switchable.value = false;
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

        // document.removeEventListener('pointerdown', unlockAudio);
        document.removeEventListener('keydown', unlockAudio);
    }

    // document.addEventListener('pointerdown', unlockAudio, eventOpts);
    document.addEventListener('keydown', unlockAudio, eventOpts);
    document.addEventListener(
        'keydown',
        (e) => {
            const keys = ['Enter', 'ArrowRight', 'ArrowLeft', 'ArrowUp', 'ArrowDown', ' ', 'Escape'];
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

    game.controls = new OrbitControls(camera, renderer.domElement);
    game.controls.enableDamping = true;
    game.controls.dampingFactor = 0.08;

    // game.controls.addEventListener('change', () => {
    //     console.log(camera.position);
    // });

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

    // -- Earth --
    const earthTexture = textureLoader.load('./earthmap1k.jpg');
    // const earthDisplacementMap = textureLoader.load(displacementURL);
    const earthGeo = new THREE.SphereGeometry(3, 64, 64);
    const earthMesh = new THREE.MeshPhongMaterial({
        color: 0xaaafff,
        map: earthTexture,
        bumpMap: displacementMap,
        // emissive: 1,
        // transparent: false,
        emissive: 0x000333,
        emissiveIntensity: 0.1,
        shininess: 20,
        // transparent: true,
        // opacity: 0.95,
    });

    earth = new THREE.Mesh(earthGeo, earthMesh);
    earth.position.x = -50.18;
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
        positions[i] = (Math.random() - 0.5) * 200;
    }

    starGeo.setAttribute('position', new THREE.BufferAttribute(positions, 3));
    stars = new THREE.Points(
        starGeo,
        new THREE.PointsMaterial({
            color: 0xffffff,
            size: 0.1,
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
            loadingBar.value = (xhr.loaded / xhr.total) * 100;
            // console.log(loadingBar.value + '% loaded');

            if ((xhr.loaded / xhr.total) * 100 === 100) {
                state.game = GAME_STATE.active;
            }
        },
        function (error) {
            console.error('An error occurred loading the GLB:', error);
        },
    );
}

function onResize() {
    const canvas = canvasRef.value;
    if (!canvas) {
        return;
    }

    const W = canvas.clientWidth;
    const H = canvas.clientHeight;

    camera.aspect = W / H;
    camera.updateProjectionMatrix();
    renderer.setSize(W, H);
}

const currentLookAt = new THREE.Vector3();

function handleCameraSwitch(target = { loc: '', camera: '' }) {
    if (!moving) {
        return;
    }

    camera.position.lerp(target.camera, 0.05); // smooth factor (0–1)
    currentLookAt.lerp(target.loc, 0.05);
    camera.lookAt(currentLookAt);

    const positionSwitchable = camera.position.distanceTo(target.camera) < 0.8;
    const lookSwitchable = currentLookAt.distanceTo(target.loc) < 0.8;
    const positionDone = camera.position.distanceTo(target.camera) < 0.001;
    const lookDone = currentLookAt.distanceTo(target.loc) < 0.001;

    switchable.value = lookSwitchable;

    if (positionDone && lookDone) {
        game.controls.target.copy(target.loc);
        game.controls.update();
        camera.position.copy(target.camera);
        moving = false;
    }
}
let bobTime = 0;
let yawTime = 0;
const bobBase = 0.2;
function handleSpaceshipSwitch(scene) {
    if (!spaceship) {
        return;
    }

    if ([SCENES.moon, SCENES.space, SCENES.sun].includes(scene)) {
        // Update orbit angle
        orbitAngle += orbitSpeed * 10;

        // Calculate new position
        const radians = orbitAngle * (Math.PI / 180);
        spaceship.position.x = Math.cos(radians) * orbitRadius;
        spaceship.position.z = Math.sin(radians) * orbitRadius;

        // Make spaceship face direction of travel
        spaceship.rotation.y = bobBase + -radians + Math.PI / 2;
        spaceship.rotation.z = 0;
    } else if (scene === SCENES.earth) {
        const yawBase = 0.3;
        spaceship.position.set(-2, 0, 2);
        bobTime += 0.08; // speed of bobbing
        yawTime += 0.02;

        const bobHeight = Math.sin(bobTime) * 0.01; // amplitude
        const yaw = Math.cos(yawTime) * 0.08; // amplitude
        spaceship.position.y = bobBase + bobHeight;
        spaceship.rotation.y = yawBase + yaw;
        spaceship.rotation.z = 0;
    } else if (scene === SCENES.sun) {
        const bobBase = 1.2;
        spaceship.position.set(3.5, 0, -5);
        bobTime += 0.08; // speed of bobbing
        yawTime += 0.05;

        const bobHeight = Math.sin(bobTime) * 0.001; // amplitude
        const yaw = Math.cos(yawTime) * -0.05; // amplitude
        spaceship.position.y = bobBase + bobHeight;
        spaceship.rotation.y = 2;
        spaceship.rotation.z = -0.51;
    }
}

const TARGETS = {
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
    space: {
        camera: new THREE.Vector3(0, 0, 10),
        loc: new THREE.Vector3(0, 0, 0),
    },
};

function animate() {
    animationId = requestAnimationFrame(animate);
    game.controls.update();

    handleCameraSwitch(TARGETS[state.scene]);
    handleSpaceshipSwitch(state.scene);

    globe.rotation.y += 0.0025;
    earth.rotation.y += 0.0012;
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
        audioListener.context?.suspend();
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
    height: 300px;
    overflow: hidden;
    bottom: 2rem;
    left: 2rem;
    font-size: 0.7rem;
    letter-spacing: 0.25em;
    color: rgba(0, 200, 255, 0.5);
    display: flex;
    align-items: flex-end;
    gap: 0.5rem;
    user-select: none;

    .dot {
        width: 6px;
        height: 6px;
        border-radius: 50%;
        background: #00e5ff;
        box-shadow: 0 0 6px #00e5ff;
        animation: pulse 2s ease-in-out infinite;
        /* display: flex; */
        margin-inline: 0.1rem;
    }

    .msg {
        /* white-space: pre-wrap; */
        display: flex;
        align-items: center;
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

.onscreen {
    position: absolute;
    color: white;
    font-size: 1rem;
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

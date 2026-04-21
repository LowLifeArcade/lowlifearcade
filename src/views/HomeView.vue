<template>
    <div class="globe-wrapper">
        <canvas
            ref="canvasRef"
            class="globe-canvas"
        />

        <div class="terminal-display">
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
                        class="cursor"
                    />
                </li>
            </ul>
        </div>
        <div
            v-if="hudActive"
            class="middle"
        >
            <h3>Controls</h3>
            <p>
                <span>wasd or&nbsp;<Arrow />:</span> <span>navigation</span>
            </p>
            <p><span>m:</span> <span>toggle sound</span></p>
            <p><span>r:</span> <span>toggle resume</span></p>
        </div>
        <div class="onscreen">
            <Transition
                name="fade"
                mode="out-in"
            >
                <div
                    v-if="!hudActive"
                    key="one"
                    class="start"
                >
                    Press Enter
                </div>
                <div
                    v-else-if="isScene(SCENES.moon)"
                    key="two"
                    class="moon"
                >
                    <div class="bg"></div>
                    <div class="content">
                        <h1>
                            work history
                            <span>
                                <a
                                    href="public/sonny_brown_resume.docx"
                                    download="sonny_brown_resume.docx"
                                >
                                    resume <Download />
                                </a>
                            </span>
                        </h1>
                        <ul class="main resume">
                            <li>
                                <span>LowLifeArcade Inc</span>
                                <span
                                    ><span class="position">engineer | </span
                                    ><span class="dates">Mar 2026 - Current&nbsp;</span></span
                                >
                            </li>
                            <li>
                                <span>Discogs Inc</span>
                                <span
                                    ><span class="position">engineer | </span
                                    ><span class="dates">Nov 2025 - Mar 2026</span></span
                                >
                            </li>
                            <li>
                                <span>PeopleFinders Inc</span>
                                <span
                                    ><span class="position">engineer | </span
                                    ><span class="dates">Apr 2023 - Nov 2025</span></span
                                >
                            </li>
                            <li>
                                <span>MHF Foundation</span
                                ><span
                                    ><span class="position">engineer | </span
                                    ><span class="dates">Jun 2022 - Mar 2023</span></span
                                >
                            </li>
                            <li>
                                <span>IT Contractor</span>
                                <span
                                    ><span class="position">engineer | </span
                                    ><span class="dates">Mar 2020 - Jun 2022</span></span
                                >
                            </li>
                            <li>
                                <span>Filmmaker</span>
                                <span
                                    ><span class="position">cinematographer | </span
                                    ><span class="dates">Jan 2015 - Dec 2020</span></span
                                >
                            </li>
                            <li>
                                <span>LootCrate</span>
                                <span>
                                    <span class="position">data warehouse | </span
                                    ><span class="dates">Aug 2013 - Jan 2015</span></span
                                >
                            </li>
                            <li>
                                <span>HappyHoodieFriends</span>
                                <span
                                    ><span class="position">engineer | </span
                                    ><span class="dates">Jan 2009 - Jun 2013</span></span
                                >
                            </li>
                        </ul>
                    </div>
                </div>
                <div
                    v-else-if="isScene(SCENES.sun)"
                    key="three"
                    class="sun"
                >
                    <div class="bg"></div>
                    <div class="content">
                        <h1>showcase</h1>
                        <ul class="main">
                            <li>
                                <a
                                    href="https://census.moonbeam.workers.dev/"
                                    target="_blank"
                                    >Moonbeam API
                                    <img
                                        src="/public/link.svg"
                                        height="14"
                                        alt="link"
                                /></a>
                                <p class="desc">API serving large US census datasets</p>
                                <ul>
                                    <li>Custom data graph system</li>
                                    <li>~1m+ daily requests</li>
                                    <li>~5ms queries</li>
                                    <li>~90ms wall time</li>
                                </ul>
                            </li>
                            <li>
                                <a
                                    href="https://www.findaneighborhood.com/"
                                    target="_blank"
                                    >FindANeighborhood
                                    <img
                                        src="/public/link.svg"
                                        height="14"
                                        alt="link"
                                /></a>
                                <p class="desc">Web app serving MLS real estate data</p>
                                <ul>
                                    <li>Six figure revenue in first year</li>
                                    <li>Custom production libraries</li>
                                </ul>
                            </li>
                            <li>
                                <a
                                    href="https://www.phonebooks.com/"
                                    target="_blank"
                                    >Phonebooks
                                    <img
                                        src="/public/link.svg"
                                        height="14"
                                        alt="link"
                                /></a>
                                <p class="desc">Web app serving large business datasets</p>
                                <ul>
                                    <li>Millions of business listenings</li>
                                </ul>
                            </li>
                        </ul>
                    </div>
                </div>
                <div
                    v-else-if="isScene(SCENES.earth)"
                    key="four"
                    class="earth"
                >
                    <div class="bg"></div>
                    <div class="content">
                        <h1>mission</h1>
                        <p>
                            To apply engineering toward solving essential human problems in a
                            rapidly changing world.
                        </p>
                        <h3>Technologies</h3>
                        <ul class="main">
                            <li>
                                Cloud
                                <ul>
                                    <li>Cloudflare: Workers / Durable Objects</li>
                                    <li>AWS: RDS / EC2 / S3</li>
                                </ul>
                            </li>
                            <li>
                                Web
                                <ul>
                                    <li>Node / Deno / Bun</li>
                                    <li>Vue / Nuxt</li>
                                    <li>React / Next / TanStack</li>
                                    <li>Three.js</li>
                                </ul>
                            </li>
                            <li>
                                Data
                                <ul>
                                    <li>ElasticSearch / ELK Stack</li>
                                    <li>SQL / NoSQL</li>
                                </ul>
                            </li>
                        </ul>
                    </div>
                </div>
            </Transition>
        </div>
        <div
            v-if="hudActive"
            class="hud"
        >
            <ul>
                <li @click="showResume = !showResume">{{ showResume ? 'on' : 'off' }} :resume</li>
                <li>{{ state.scene }} :scene</li>
                <li class="music">
                    <div @click="onToggleSound">
                        {{ state.audio === AL_STATE.running ? 'on' : 'off' }} :sound
                    </div>
                </li>
                <li class="credits">
                    music by
                    <a
                        href="https://pixabay.com/users/idoberg-34953295/"
                        target="_blank"
                        >IdoBerg</a
                    >
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
import { MSGS } from '../consts/msgs';

import Arrow from '@/components/arrow.vue';
import Download from '@/components/download.vue';

const ALLOWED_KEYS = [
    'Enter',
    'ArrowRight',
    'ArrowLeft',
    'ArrowUp',
    'ArrowDown',
    ' ', // spacebar
    'Escape',
    'q',
    'w',
    'a',
    's',
    'd',
];
const NAV_KEYS = {
    forward: ['Enter', 'ArrowRight', 'ArrowUp', ' ', 'w', 'd'],
    backward: ['ArrowLeft', 'ArrowDown', 'a', 's'],
    exit: ['Escape', 'q'],
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
const showResume = ref(true);
const isScene = (scene) => showResume.value && hudActive.value && state.scene === scene;

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
    initActionTaken = false,
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

document.addEventListener(
    'keydown',
    (e) => {
        if (['m'].includes(e.key)) {
            onToggleSound();
        }

        if (['r'].includes(e.key)) {
            showResume.value = !showResume.value;
        }
    },
    eventOpts,
);

function toggleState(key) {
    if (NAV_KEYS.exit.includes(key)) {
        state.scene = SCENES.space;
        moving = true;
        switchable.value = false;

        if (audioListener?.context?.state === 'running') {
            audioListener?.context.suspend();
        }

        return;
    }

    if (!game.controls.enabled && key !== 'Enter') {
        return;
    }

    if (state.audio === AL_STATE.running && audioListener?.context?.state === 'suspended') {
        audioListener?.context?.resume();
    }

    const forward = NAV_KEYS.forward.includes(key);
    const back = NAV_KEYS.backward.includes(key);

    handleDirection({ forward, back });
}

function handleDirection({ forward, back }) {
    if (state.scene === SCENES.moon) {
        if (forward) {
            state.scene = SCENES.sun;
        } else if (back) {
            state.scene = SCENES.earth;
        }

        moving = true;
        switchable.value = false;
    } else if (state.scene === SCENES.sun) {
        if (forward) {
            state.scene = SCENES.earth;
        } else if (back) {
            state.scene = SCENES.moon;
        }

        moving = true;
        switchable.value = false;
    } else if (state.scene === SCENES.earth) {
        if (forward) {
            state.scene = SCENES.moon;
        } else if (back) {
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

    function initAction(e) {
        if (e?.key && !ALLOWED_KEYS.includes(e?.key)) {
            return;
        }

        if (audioListener?.context?.state === 'suspended') {
            audioListener?.context.resume();
        }

        // if pointerdown change scene
        if (!e?.key) {
            state.scene = SCENES.moon;
        }

        document.removeEventListener('mousedown', initAction);
        document.removeEventListener('keydown', initAction);
        initActionTaken = true;
    }

    document.addEventListener('mousedown', initAction, eventOpts);
    document.addEventListener('keydown', initAction, eventOpts);
    document.addEventListener(
        'keydown',
        (e) => {
            if (!ALLOWED_KEYS.includes(e.key)) {
                return;
            }

            toggleState(e.key);
        },
        eventOpts,
    );

    let touchstartX = 0;
    let touchendX = 0;
    let touchstartY = 0;
    let touchendY = 0;

    const zone = document.getElementById('swipeZone') || document;

    zone.addEventListener(
        'touchstart',
        (e) => {
            touchstartX = e.changedTouches[0].screenX;
            touchstartY = e.changedTouches[0].screenY;
        },
        false,
    );

    zone.addEventListener(
        'touchend',
        (e) => {
            touchendX = e.changedTouches[0].screenX;
            touchendY = e.changedTouches[0].screenY;

            if (!initActionTaken) {
                initActionTaken = true;
                return;
            }

            handleGesture();
        },
        false,
    );

    function handleGesture() {
        const deltaX = touchendX - touchstartX;
        const deltaY = touchendY - touchstartY;
        const DIRS = {
            left: 'left',
            right: 'right',
            up: 'up',
            down: 'down',
        };
        let forward = true;
        let swipe;

        if (Math.abs(deltaX) > Math.abs(deltaY)) {
            if (deltaX > 0) {
                swipe = DIRS.right;
                console.log('Swiped Right');
                forward = false;
            } else {
                swipe = DIRS.left;
                forward = true;
                console.log('Swiped Left');
            }
        } else {
            if (deltaY > 0) {
                swipe = DIRS.down;
                console.log('Swiped Down');
            } else {
                // forward = true;
                swipe = DIRS.up;
                console.log('Swiped Up');
            }
        }

        if (state.audio === AL_STATE.running && audioListener?.context?.state === 'suspended') {
            audioListener?.context?.resume();
        }

        if (swipe === DIRS.down) {
            state.scene = SCENES.space;
            moving = true;
            switchable.value = false;

            if (audioListener?.context?.state === 'running') {
                audioListener?.context.suspend();
            }

            return;
        }

        handleDirection({ forward, back: !forward });
    }

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
    const earthTexture = textureLoader.load('./earth_terrain.jpg');
    const earthDisplacementMap = textureLoader.load('./earth_terrain.jpg');
    const earthGeo = new THREE.SphereGeometry(3, 64, 64);
    const earthMesh = new THREE.MeshPhongMaterial({
        color: 0xcccfff,
        map: earthTexture,
        bumpMap: earthDisplacementMap,
        emissive: 1,
        bumpScale: 1.1,
        emissive: 0x000333,
        emissiveIntensity: 0.1,
        shininess: 20,
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

.fade-enter-active,
.fade-leave-active {
    transition: opacity 1s ease-in-out;
}

.fade-enter-from,
.fade-leave-to {
    opacity: 0;
}

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

.globe-canvas {
    width: 100%;
    height: 100%;
    display: block;
}

.terminal-display {
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

    .cursor {
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
        display: flex;
        align-items: center;
    }

    ul {
        margin: unset;
        padding: unset;
    }
}

.hud {
    font-size: 1rem;
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
    }

    .credits {
        margin-top: 0.4rem;
        font-size: 0.8rem;
    }
}

.middle {
    position: absolute;
    bottom: 2rem;
    /* color: white; */
    color: rgb(74, 119, 139);
    min-width: 230px;

    h3 {
        text-align: center;
        margin-bottom: .3rem;
    }

    p {
        display: flex;
        justify-content: space-between;
        align-items: center;
        line-height: 1.5rem;

        span {
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
    }

    @media (width < 500px) {
        display: none;
    }
}

.onscreen {
    position: absolute;
    color: #fefefe;
    color: #dddddd;
    font-size: 1rem;
    padding: 2rem;

    .bg {
        /* background: radial-gradient(circle at center, rgba(0, 0, 0, 0.695) 56%, transparent 92%); */
        background: radial-gradient(rgba(0, 0, 0, 0.7) 10.6%, transparent 87.5%);
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        opacity: 1;
    }

    .content {
        position: relative;
        z-index: 1;
        display: flex;
        flex-direction: column;
        gap: 1rem;

        h1 {
            white-space: nowrap;
        }

        ul.main {
            list-style: none;
            margin: unset;
            padding: unset;
            gap: 1rem;
            display: grid;

            &.resume {
                li {
                    display: flex;
                    justify-content: space-between;
                    gap: 2rem;

                    span {
                        white-space: nowrap;
                    }

                    .dates {
                        /* margin-left: .5rem; */
                    }
                }
            }
        }

        a {
            font-size: 1.2rem;
            display: inline-block;
            margin-bottom: 0.6rem;
        }

        .desc {
            /* margin-block: 0.75rem; */
            /* font-size: .9rem; */
            padding: 0.5rem;
        }
    }

    .start {
        font-size: 3rem;
    }

    &:has(.sun) {
        right: 10%;
        display: flex;
        justify-content: flex-end;

        .sun {
            max-width: 400px;
        }
    }

    &:has(.moon) {
        margin-left: 20rem;
        /* max-width: 380px; */

        h1 {
            display: flex;
            justify-content: space-between;
            align-items: center;

            a {
                display: flex;
                align-items: center;
                gap: .3rem;
                margin: unset;
            }
        }
    }

    &:has(.earth) {
        margin-left: 10rem;
        top: 16%;

        .earth {
            max-width: 450px;
        }
    }

    @media (width < 500px) {
        margin: unset !important;

        .position {
            display: none;
        }
    }
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

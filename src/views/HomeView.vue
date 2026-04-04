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

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue';
import * as THREE from 'three';

const canvasRef = ref(null);
let animationId = null;
let renderer, scene, camera, globe, atmosphere, stars;

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

    // ── Globe ──────────────────────────────────────────────
    const geo = new THREE.SphereGeometry(1, 64, 64);

    // Wireframe overlay mesh (latlon lines)
    const wireMat = new THREE.MeshBasicMaterial({
        color: 0x00e5ff,
        wireframe: true,
        transparent: true,
        opacity: 0.08,
    });
    const wireGlobe = new THREE.Mesh(geo, wireMat);
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
    globe = new THREE.Mesh(geo, mat);
    scene.add(globe);

    // ── Atmosphere glow (slightly larger sphere) ───────────
    const atmGeo = new THREE.SphereGeometry(1.12, 64, 64);
    const atmMat = new THREE.MeshPhongMaterial({
        color: 0x00aaff,
        transparent: true,
        opacity: 0.06,
        side: THREE.FrontSide,
    });
    atmosphere = new THREE.Mesh(atmGeo, atmMat);
    // scene.add(atmosphere);

    // ── Stars ──────────────────────────────────────────────
    const starGeo = new THREE.BufferGeometry();
    const starCount = 1800;
    const positions = new Float32Array(starCount * 3);
    for (let i = 0; i < starCount * 3; i++) {
        positions[i] = (Math.random() - 0.5) * 60;
    }
    starGeo.setAttribute('position', new THREE.BufferAttribute(positions, 3));
    stars = new THREE.Points(starGeo, new THREE.PointsMaterial({ color: 0xffffff, size: 0.06, transparent: true, opacity: 0.7 }));
    scene.add(stars);

    // ── Lights ─────────────────────────────────────────────
    // scene.add(new THREE.AmbientLight(0x1a3a6e, 1.2));

    const sun = new THREE.DirectionalLight(0x88ccff, 2.5);
    sun.position.set(4, 2, -6);
    scene.add(sun);

    const rim = new THREE.DirectionalLight(0x004488, 1.0);
    rim.position.set(-3, -1, -2);
    scene.add(rim);

    // ── Resize ─────────────────────────────────────────────
    window.addEventListener('resize', onResize);
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
    globe.rotation.y += 0.0025;
    atmosphere.rotation.y += 0.0022;
    stars.rotation.y += 0.0002;
    renderer.render(scene, camera);
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
    background: radial-gradient(ellipse at 60% 40%, #071428 0%, #020810 70%);
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
    background: radial-gradient(circle, rgba(0, 160, 255, 0.12) 0%, transparent 70%);
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

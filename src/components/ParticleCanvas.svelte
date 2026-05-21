<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import * as THREE from 'three';

  let canvas: HTMLCanvasElement;
  let renderer: THREE.WebGLRenderer;
  let animId: number;

  onMount(() => {
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(55, window.innerWidth / window.innerHeight, 0.1, 100);
    camera.position.z = 5;

    renderer = new THREE.WebGLRenderer({ canvas, antialias: true, alpha: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

    const count = 2500;
    const pos = new Float32Array(count * 3);
    const col = new Float32Array(count * 3);
    for (let i = 0; i < count; i++) {
      pos[i*3]   = (Math.random() - 0.5) * 40;
      pos[i*3+1] = (Math.random() - 0.5) * 40;
      pos[i*3+2] = (Math.random() - 0.5) * 40;
      const t = Math.random();
      if (t < 0.65) {
        col[i*3] = 0.7 + Math.random()*0.3; col[i*3+1] = 0.8 + Math.random()*0.2; col[i*3+2] = 1.0;
      } else {
        col[i*3] = 0.0; col[i*3+1] = 0.6 + Math.random()*0.4; col[i*3+2] = 1.0;
      }
    }
    const geo = new THREE.BufferGeometry();
    geo.setAttribute('position', new THREE.BufferAttribute(pos, 3));
    geo.setAttribute('color', new THREE.BufferAttribute(col, 3));
    const stars = new THREE.Points(geo, new THREE.PointsMaterial({ size: 0.045, vertexColors: true, transparent: true, opacity: 0.7 }));
    scene.add(stars);

    // Faint distant brain outline
    const bGeo = new THREE.IcosahedronGeometry(0.6, 3);
    const bWire = new THREE.LineSegments(
      new THREE.WireframeGeometry(bGeo),
      new THREE.LineBasicMaterial({ color: 0x00d4ff, transparent: true, opacity: 0.06 })
    );
    bWire.position.set(3, 0.5, -3);
    scene.add(bWire);

    let t = 0;
    const animate = () => {
      animId = requestAnimationFrame(animate);
      t += 0.002;
      stars.rotation.y = t * 0.015;
      stars.rotation.x = t * 0.008;
      bWire.rotation.y = t * 0.3;
      bWire.rotation.x = t * 0.15;
      renderer.render(scene, camera);
    };
    animate();

    const onResize = () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    };
    window.addEventListener('resize', onResize);
    return () => window.removeEventListener('resize', onResize);
  });

  onDestroy(() => {
    cancelAnimationFrame(animId);
    renderer?.dispose();
  });
</script>

<canvas bind:this={canvas} class="particle-canvas" />

<style>
  .particle-canvas {
    position: fixed;
    inset: 0;
    width: 100%;
    height: 100%;
    z-index: 0;
    pointer-events: none;
  }
</style>

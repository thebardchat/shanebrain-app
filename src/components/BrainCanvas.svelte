<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import * as THREE from 'three';

  let canvas: HTMLCanvasElement;
  let renderer: THREE.WebGLRenderer;
  let animId: number;

  onMount(() => {
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(55, window.innerWidth / window.innerHeight, 0.1, 200);
    camera.position.z = 3.5;

    renderer = new THREE.WebGLRenderer({ canvas, antialias: true, alpha: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

    // ── Star field ──
    const starCount = 6000;
    const starPos = new Float32Array(starCount * 3);
    const starCol = new Float32Array(starCount * 3);
    for (let i = 0; i < starCount; i++) {
      const theta = Math.random() * Math.PI * 2;
      const phi = Math.acos(2 * Math.random() - 1);
      const r = 15 + Math.random() * 60;
      starPos[i*3]   = r * Math.sin(phi) * Math.cos(theta);
      starPos[i*3+1] = r * Math.sin(phi) * Math.sin(theta);
      starPos[i*3+2] = r * Math.cos(phi);
      const t = Math.random();
      if (t < 0.55) {
        starCol[i*3] = 0.75 + Math.random()*0.25;
        starCol[i*3+1] = 0.8 + Math.random()*0.2;
        starCol[i*3+2] = 1.0;
      } else if (t < 0.8) {
        starCol[i*3] = 0.0; starCol[i*3+1] = 0.7 + Math.random()*0.3; starCol[i*3+2] = 1.0;
      } else {
        starCol[i*3] = 0.5 + Math.random()*0.3; starCol[i*3+1] = 0.05; starCol[i*3+2] = 0.85 + Math.random()*0.15;
      }
    }
    const starGeo = new THREE.BufferGeometry();
    starGeo.setAttribute('position', new THREE.BufferAttribute(starPos, 3));
    starGeo.setAttribute('color', new THREE.BufferAttribute(starCol, 3));
    const stars = new THREE.Points(starGeo, new THREE.PointsMaterial({ size: 0.06, vertexColors: true, transparent: true, opacity: 0.85 }));
    scene.add(stars);

    // ── Nebula backdrop ──
    const nebulaGeo = new THREE.SphereGeometry(12, 32, 32);
    scene.add(new THREE.Mesh(nebulaGeo, new THREE.MeshBasicMaterial({ color: 0x1a0a40, transparent: true, opacity: 0.18, side: THREE.BackSide })));

    // ── Brain mesh (icosahedron) ──
    const brainGeo = new THREE.IcosahedronGeometry(1, 5);

    // Wireframe shell
    const wireGeo = new THREE.WireframeGeometry(brainGeo);
    const wireMat = new THREE.LineBasicMaterial({ color: 0x00d4ff, transparent: true, opacity: 0.14 });
    const wireMesh = new THREE.LineSegments(wireGeo, wireMat);

    // Point cloud on surface
    const ptsMat = new THREE.PointsMaterial({ color: 0x00d4ff, size: 0.022, transparent: true, opacity: 0.9 });
    const ptsMesh = new THREE.Points(brainGeo, ptsMat);

    // Inner glow sphere
    const glowGeo = new THREE.SphereGeometry(0.92, 32, 32);
    const glowMat = new THREE.MeshBasicMaterial({ color: 0x0066aa, transparent: true, opacity: 0.06 });
    const glowMesh = new THREE.Mesh(glowGeo, glowMat);

    // Outer aura
    const auraMat = new THREE.MeshBasicMaterial({ color: 0x7c3aed, transparent: true, opacity: 0.04, side: THREE.BackSide });
    const auraMesh = new THREE.Mesh(new THREE.SphereGeometry(1.45, 32, 32), auraMat);

    const brain = new THREE.Group();
    brain.add(wireMesh, ptsMesh, glowMesh, auraMesh);
    scene.add(brain);

    // ── Mouse parallax ──
    let mx = 0, my = 0;
    const onMouseMove = (e: MouseEvent) => {
      mx = (e.clientX / window.innerWidth - 0.5) * 2;
      my = (e.clientY / window.innerHeight - 0.5) * 2;
    };
    window.addEventListener('mousemove', onMouseMove);

    let t = 0;
    const animate = () => {
      animId = requestAnimationFrame(animate);
      t += 0.004;

      brain.rotation.y = t * 0.25 + mx * 0.15;
      brain.rotation.x = Math.sin(t * 0.08) * 0.18 - my * 0.1;

      const pulse = 1 + Math.sin(t * 2.2) * 0.018;
      brain.scale.setScalar(pulse);

      wireMat.opacity = 0.10 + Math.sin(t * 1.8) * 0.05;

      stars.rotation.y = t * 0.008;
      stars.rotation.x = t * 0.004;

      renderer.render(scene, camera);
    };
    animate();

    const onResize = () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    };
    window.addEventListener('resize', onResize);

    return () => {
      window.removeEventListener('mousemove', onMouseMove);
      window.removeEventListener('resize', onResize);
    };
  });

  onDestroy(() => {
    cancelAnimationFrame(animId);
    renderer?.dispose();
  });
</script>

<canvas bind:this={canvas} class="brain-canvas" />

<style>
  .brain-canvas {
    position: fixed;
    inset: 0;
    width: 100%;
    height: 100%;
    z-index: 0;
    pointer-events: none;
  }
</style>

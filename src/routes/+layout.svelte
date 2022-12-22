<script lang="ts">
  import { onMount } from "svelte";
  import {
    BoxGeometry,
    Mesh,
    PerspectiveCamera,
    Raycaster,
    Scene,
    Vector2,
    WebGLRenderer,
  } from "three";
  import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
  import "./styles.scss";

  let canvas: HTMLCanvasElement,
    width = 0,
    height = 0,
    requestID = 0;

  const scene = new Scene(),
    camera = new PerspectiveCamera(50, 2, 1, 1000),
    mouse = new Vector2(),
    raycaster = new Raycaster();

  camera.position.set(0, 0, 200);
  scene.add(new Mesh(new BoxGeometry(50, 50, 50, 50, 50, 50)));

  onMount(() => {
    const renderer = new WebGLRenderer({ canvas, antialias: false });
    const controls = new OrbitControls(camera, canvas);
    Object.assign(controls, {
      enableDamping: true,
      enableRotate: true,
      rotateSpeed: 0.5,
      enablePan: false,
      enableZoom: true,
      minDistance: 50 * 1.1,
      maxDistance: camera.far - 50,
    });

    const resize = () => {
      camera.aspect = width / height;
      camera.updateProjectionMatrix();
      renderer.setSize(width, height);
      renderer.setPixelRatio(devicePixelRatio);
    };

    const animate = () => {
      controls.update();
      renderer.render(scene, camera);
      requestID = requestAnimationFrame(animate);
    };

    resize();
    addEventListener("resize", resize);
    requestID = requestAnimationFrame(animate);
    return () => {
      removeEventListener("resize", resize);
      cancelAnimationFrame(requestID);
    };
  });
</script>

<svelte:window bind:innerWidth={width} bind:innerHeight={height} />

<header>
  <nav>
    <a href="/">HOME</a>
  </nav>
</header>
<main>
  <canvas
    bind:this={canvas}
    on:mousemove={(e) =>
      mouse.set((e.clientX / width) * 2 - 1, -(e.clientY / height) * 2 + 1)}
    on:mousedown={() => {
      raycaster.setFromCamera(mouse, camera);
      let hit;
      if (hit) {
      }
    }}
  />
</main>
<footer>
  <small>© Jason Karjadi 2022. All rights reserved</small>
</footer>
<slot />

<style lang="scss">
  header,
  footer {
    position: absolute;
    left: 0;
    right: 0;
    height: 3rem;
    background-color: rgba($color: #00ffff, $alpha: 0.2);
    display: flex;
    align-items: center;
    padding: 0 3rem;
    z-index: 2;
  }

  header {
    top: 0;
  }

  footer {
    bottom: 0;
  }
</style>

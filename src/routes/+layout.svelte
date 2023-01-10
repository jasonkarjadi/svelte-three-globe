<script lang="ts">
  import { onMount } from "svelte";
  import {
    AmbientLight,
    DirectionalLight,
    PerspectiveCamera,
    Raycaster,
    Scene,
    Vector2,
    WebGLRenderer,
  } from "three";
  import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
  import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
  import "./styles.scss";

  let canvas: HTMLCanvasElement,
    width = 0,
    height = 0,
    requestID = 0;

  const scene = new Scene(),
    camera = new PerspectiveCamera(50, 2, 1, 5000),
    mouse = new Vector2(),
    raycaster = new Raycaster(),
    gltfLoader = new GLTFLoader(),
    directionalLight = new DirectionalLight(0xffffff, 2),
    ambientLight = new AmbientLight(0x404040, 2);

  camera.position.set(0, 0, 1500);
  directionalLight.position.y += 1000;
  scene.add(camera.add(directionalLight), ambientLight);

  onMount(() => {
    const renderer = new WebGLRenderer({ canvas, antialias: false });
    const controls = new OrbitControls(camera, canvas);
    Object.assign(controls, {
      enableDamping: true,
      enableRotate: true,
      rotateSpeed: 0.5,
      enablePan: false,
      enableZoom: true,
      minDistance: 800,
      maxDistance: camera.far - 1000,
    });
    gltfLoader.load(
      "/earth.glb",
      (gltf) => {
        gltf.scene.children[0].rotateY(4);
        scene.add(gltf.scene);
      },
      (xhr) => {
        // console.log(`${(xhr.loaded / xhr.total) * 100}% loaded`);
      },
      (err) => {
        console.error(err);
      }
    );

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

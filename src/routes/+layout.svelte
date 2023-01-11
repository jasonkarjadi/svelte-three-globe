<script lang="ts">
  import { onMount } from "svelte";
  import {
    AmbientLight,
    Color,
    DirectionalLight,
    PerspectiveCamera,
    Raycaster,
    Scene,
    WebGLRenderer,
  } from "three";
  import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
  import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
  import "./styles.scss";

  let canvas: HTMLCanvasElement,
    innerWidth = 0,
    innerHeight = 0,
    requestID = 0,
    offsetWidth = 32,
    offsetHeight = 120;

  const scene = new Scene(),
    camera = new PerspectiveCamera(50, 2, 1, 5000),
    raycaster = new Raycaster(),
    gltfLoader = new GLTFLoader(),
    directionalLight = new DirectionalLight(0xffffff, 2),
    ambientLight = new AmbientLight(0xedf2f7, 1);

  camera.position.setZ(2000);
  directionalLight.position.y += 800;
  scene.background = new Color(0x0e0e10);
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
      camera.aspect = (innerWidth - offsetWidth) / (innerHeight - offsetHeight);
      camera.updateProjectionMatrix();
      renderer.setSize(innerWidth - offsetWidth, innerHeight - offsetHeight);
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

<svelte:window bind:innerWidth bind:innerHeight />

<header>
  <nav class="topleft">
    <button><i class="fa-solid fa-bars" /></button>
  </nav>
  <div class="topmiddle" />
  <div class="topright"><button><i class="fa-solid fa-gear" /></button></div>
</header>
<main>
  <canvas bind:this={canvas} />
</main>
<slot />
<footer>
  <button on:click={() => {}}>
    {#if false}
      Open
    {:else}
      <i class="fa-solid fa-rectangle-xmark" />
      Close
    {/if}
  </button>
</footer>

<style lang="scss">
  $my-border: solid #2d3748 1px;
  $color-jetblack: #0e0e10;

  header,
  footer {
    height: 3rem;
    display: flex;
  }
  header {
    > .topleft {
      button {
        border-right: $my-border;
        border-radius: 0 0 1rem 0;
      }
    }

    > .topmiddle {
      flex: 1;
    }

    > .topright {
      button {
        border-radius: 0 0 0 1rem;
        border-left: $my-border;
      }
    }

    button {
      height: 100%;
      width: 4rem;
      box-shadow: 0 2px 5px $color-jetblack;
      border-bottom: $my-border;
    }
  }
  footer {
    > button {
      flex: 1;
      border-radius: 1rem 1rem 0 0;
      box-shadow: 0 -2px 5px $color-jetblack;
      border-top: $my-border;
    }
  }
  canvas {
    margin: 1rem auto 0.5rem;
    border-radius: 1rem;
    box-shadow: 0 0 5px 2px $color-jetblack;
    border: $my-border;
  }
</style>

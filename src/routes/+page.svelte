<script lang="ts">
  import { onMount } from "svelte";
  import * as THREE from "three";
  import { FlyControls } from "three/examples/jsm/controls/FlyControls";

  let canvas: HTMLElement;

  const tunnelPath = new THREE.CatmullRomCurve3([
    new THREE.Vector3(0, 0, 0),
    new THREE.Vector3(200, 50, 0),
    new THREE.Vector3(400, 100, 50),
    new THREE.Vector3(600, 300, 200),
    new THREE.Vector3(1000, 500, 200),
    new THREE.Vector3(1200, 300, 200),
    new THREE.Vector3(1500, 200, 400),
    new THREE.Vector3(1800, 500, 800),
  ]);

  onMount(() => {
    const renderer = new THREE.WebGLRenderer({ antialias: true, canvas });
    const fov = 75;
    const aspect = 1; // the canvas default
    const near = 0.1;
    const far = 1000;
    const camera = new THREE.PerspectiveCamera(fov, aspect, near, far);

    const geometry = new THREE.TubeGeometry(tunnelPath, 128, 100, 16, false);

    let spaceTexture = new THREE.TextureLoader().load("space.jpg");
    spaceTexture.wrapT = THREE.MirroredRepeatWrapping;
    spaceTexture.wrapS = THREE.MirroredRepeatWrapping;

    spaceTexture.repeat.set(8, 2);

    const material = new THREE.MeshPhongMaterial({
      color: 0x333333,
      side: THREE.BackSide, // TODO: change to BackSide on production
      map: spaceTexture,
    });
    let tunnel = new THREE.Mesh(geometry, material);

    const scene = new THREE.Scene();
    scene.add(tunnel);

    const light = new THREE.PointLight(0xffffff, 10, 600, 2); // TODO: Change to DirectionalLight if performance is bad
    light.position.set(camera.position.x, camera.position.y, camera.position.z);
    scene.add(light);

    renderer.render(scene, camera);

    // Uncomment for manual camera control
    // let controls = new FlyControls(camera, renderer.domElement);
    // controls.movementSpeed = 200;
    // controls.rollSpeed = Math.PI / 24;
    // controls.autoForward = false;
    // controls.dragToLook = true;

    function render(time: number) {
      // TODO:  use scroll position for this
      // animation frame index : value between 0 and 1
      let afi = (time * 0.0001) % 0.8;

      let p1 = tunnelPath.getPointAt(afi);
      let p2 = tunnelPath.getPointAt(Math.min(afi + 0.1, 1));

      // Uncomment for manual camera control
      // controls.update(0.01);

      // Comment out for manual camera control
      camera.position.set(p1.x, p1.y, p1.z);
      camera.lookAt(p2.x, p2.y, p2.z);

      light.position.set(
        camera.position.x,
        camera.position.y,
        camera.position.z
      );

      renderer.render(scene, camera);

      requestAnimationFrame(render);
    }
    requestAnimationFrame(render);
  });
</script>

<canvas class="canvas" width="1080" height="920" bind:this={canvas} />

<style>
  .canvas {
    margin: 0px;
    padding: 0px;
    display: block;
  }
</style>

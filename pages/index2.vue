<template>
  <div>
    Index page
    <ClientOnly>
      <div ref="rendererDom"></div>
    </ClientOnly>
  </div>
</template>

<script lang="ts" setup>
definePageMeta({
  ssr: false
})
import * as THREE from 'three'

// THREEx.ArToolkitContext.baseURL = "./";

const rendererDom = ref<
  HTMLCanvasElement | HTMLDivElement | HTMLIFrameElement | null
>(null)
async function setup() {
  // log in the console the two newly created namespaces
  // console.log(THREEx);
  // console.log(ARjs)
  const { THREEx, ARjs } = await import("@ar-js-org/ar.js-threejs")

  const renderer = new THREE.WebGLRenderer({
    antialias: true,
    alpha: true
  });
  renderer.setClearColor(new THREE.Color('lightgrey'), 0)
  renderer.setSize(640, 480);
  renderer.domElement.style.position = 'absolute'
  renderer.domElement.style.top = '0px'
  renderer.domElement.style.left = '0px'
  // document.body.appendChild(renderer.domElement);
  rendererDom.value = renderer.domElement

  // array of functions for the rendering loop
  const onRenderFcts: any[] = [];
  let arToolkitContext: any, arMarkerControls;

  // init scene and camera
  const scene = new THREE.Scene();

  // Create a camera
  const camera = new THREE.PerspectiveCamera();
  scene.add(camera);


  const arToolkitSource = new THREEx.ArToolkitSource({
    // to read from the webcam
    sourceType: 'webcam',

    sourceWidth: window.innerWidth > window.innerHeight ? 640 : 480,
    sourceHeight: window.innerWidth > window.innerHeight ? 480 : 640,
  })

  arToolkitSource.init(function onReady() {
    // use a resize to fullscreen mobile devices
    setTimeout(function () {
      onResize()
    }, 1000);
  }, function onError() { })

  // handle resize
  window.addEventListener('resize', function () {
    onResize()
  })

  // listener for end loading of NFT marker
  window.addEventListener('arjs-nft-loaded', function (ev) {
    console.log(ev);
  })

  function onResize() {
    arToolkitSource.onResizeElement()
    arToolkitSource.copyElementSizeTo(renderer.domElement)
    if (arToolkitContext.arController !== null) {
      arToolkitSource.copyElementSizeTo(arToolkitContext.arController.canvas)
    }
  }

  // create atToolkitContext
  arToolkitContext = new THREEx.ArToolkitContext({
    detectionMode: 'mono',
    cameraParametersUrl: 'https://ar-js-org.github.io/AR.js/data/data/camera_para.dat',
    canvasWidth: 640,
    canvasHeight: 480,
  })

  // initialize it
  arToolkitContext.init(function onCompleted() {
    // copy projection matrix to camera
    camera.projectionMatrix.copy(arToolkitContext.getProjectionMatrix());
  })

  console.log('context done!')

  // init controls for camera
  const markerControls = new THREEx.ArMarkerControls(arToolkitContext, camera, {
    type: 'nft',//
    descriptorsUrl: 'data/dataNFT/pinball',
    changeMatrixMode: 'cameraTransformMatrix'
  })

  scene.visible = false

  const root = new THREE.Object3D();
  scene.add(root);

  const geometry = new THREE.BoxGeometry(1, 1, 1);
  const material = new THREE.MeshNormalMaterial({
    transparent: true,
    opacity: 0.5,
    side: THREE.DoubleSide
  })
  const cube = new THREE.Mesh(geometry, material);
  cube.position.y = 90;
  cube.scale.set(180, 180, 180);
  root.matrixAutoUpdate = false;
  root.add(cube);

  window.addEventListener('arjs-nft-init-data', function (nft: any) {
    console.log(nft);
    const msg = nft.detail;
    cube.position.z = -(msg.height / msg.dpi * 2.54 * 10) / 2.0; //y axis?
    cube.position.x = (msg.width / msg.dpi * 2.54 * 10) / 2.0; //x axis?
  })

  const animate = function () {
    requestAnimationFrame(animate);

    if (!arToolkitSource.ready) {
      return;
    }

    arToolkitContext.update(arToolkitSource.domElement)

    // update scene.visible if the marker is seen
    scene.visible = camera.visible;

    renderer.render(scene, camera);
  };

  requestAnimationFrame(animate);
}

onMounted(() => {
  setup()
})

</script>

<style></style>
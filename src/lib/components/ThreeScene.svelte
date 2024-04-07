<script lang="ts">
	import { onMount } from 'svelte';
	import * as THREE from 'three';
	import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
	import { OrbitControls } from 'three/examples/jsm/Addons.js';

	let scene: THREE.Scene;
	let camera: THREE.PerspectiveCamera;
	let renderer: THREE.WebGLRenderer;

	onMount(async () => {
		initScene();

		const controls = new OrbitControls(camera, renderer.domElement);

		let gadget: THREE.Group<THREE.Object3DEventMap> | null = null;
		loadGLTF('/models/music-gadget.gltf').then((mesh) => {
			gadget = mesh;
			if (gadget) {
				scene.add(gadget);
				gadget.rotation.set(0, 0, 0);
				gadget.castShadow = true;

				gadget.children.forEach((element) => {
					element.castShadow = true;
				});

				const screen = gadget.children[10] as THREE.Mesh;
				const mat = new THREE.MeshPhongMaterial({ color: 0xffffff });
				screen.material = mat;
			}
		});

		const floorGeo = new THREE.PlaneGeometry(20, 20);
		const floorMat = new THREE.MeshLambertMaterial({ color: 0xf5f5f5 });
		const floor = new THREE.Mesh(floorGeo, floorMat);
		floor.rotation.set(4.7, 0, 0);
		floor.receiveShadow = true;
		scene.add(floor);

		const directionalLight = new THREE.PointLight(0x3025fc, 15);
		directionalLight.position.set(5, 4, -3);
		directionalLight.castShadow = true;
		scene.add(directionalLight);

		directionalLight.shadow.mapSize.width = 512; // default
		directionalLight.shadow.mapSize.height = 512; // default
		directionalLight.shadow.camera.near = 0.5; // default
		directionalLight.shadow.camera.far = 500; // default

		const directionalLight2 = new THREE.PointLight(0xfc25f8, 15);
		directionalLight2.position.set(-6, 4, -2);
		directionalLight2.castShadow = true;
		scene.add(directionalLight2);

		directionalLight2.shadow.mapSize.width = 512; // default
		directionalLight2.shadow.mapSize.height = 512; // default
		directionalLight2.shadow.camera.near = 0.5; // default
		directionalLight2.shadow.camera.far = 500; // default

		// const helper = new THREE.CameraHelper(directionalLight.shadow.camera);
		// scene.add(helper);

		// animation
		function animation(time: number) {
			renderer.render(scene, camera);

			controls.update();
		}

		renderer.setAnimationLoop(animation);
	});

	////////

	function initScene() {
		const width = window.innerWidth,
			height = window.innerHeight;

		scene = new THREE.Scene();
		camera = new THREE.PerspectiveCamera(70, width / height, 0.01, 1000);
		camera.position.y = 8;
		camera.lookAt(0, 0, 0);

		//RENDERER
		renderer = new THREE.WebGLRenderer({
			antialias: true
		});
		renderer.setSize(width, height);
		renderer.shadowMap.enabled = true;
		renderer.shadowMap.type = THREE.PCFSoftShadowMap;

		document.getElementById('threeContainer')?.appendChild(renderer.domElement);
	}

	function loadGLTF(url: string): Promise<THREE.Group<THREE.Object3DEventMap> | null> {
		const loader = new GLTFLoader();

		return new Promise((resolve, reject) => {
			loader.load(
				url,
				function (gltf) {
					resolve(gltf.scene);
				},
				undefined,
				function (error) {
					console.warn('Failed to load GLTF');
					console.error(error);
					reject(null);
				}
			);
		});
	}
</script>

<div id="threeContainer"></div>

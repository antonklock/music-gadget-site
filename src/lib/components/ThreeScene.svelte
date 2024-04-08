<script lang="ts">
	import gsap from 'gsap';
	import { onMount } from 'svelte';
	import * as THREE from 'three';
	import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
	import { OrbitControls } from 'three/examples/jsm/Addons.js';

	const keySFX = [
		'https://firebasestorage.googleapis.com/v0/b/music-gadget-d82f8.appspot.com/o/SFX%2Fkeys%2Fkey_01.wav?alt=media&token=b86629d0-46cf-42ed-af0b-98ca885f2f6c',
		'https://firebasestorage.googleapis.com/v0/b/music-gadget-d82f8.appspot.com/o/SFX%2Fkeys%2Fkey_02.wav?alt=media&token=68cf8bc4-8e05-4bf9-b014-3fb0e32393fc',
		'https://firebasestorage.googleapis.com/v0/b/music-gadget-d82f8.appspot.com/o/SFX%2Fkeys%2Fkey_03.wav?alt=media&token=38fb822a-5894-4211-bc45-3f9f06bb03c6',
		'https://firebasestorage.googleapis.com/v0/b/music-gadget-d82f8.appspot.com/o/SFX%2Fkeys%2Fkey_04.wav?alt=media&token=b96ddd53-5a7a-49a5-bcc6-6714969f8414',
		'https://firebasestorage.googleapis.com/v0/b/music-gadget-d82f8.appspot.com/o/SFX%2Fkeys%2Fkey_05.wav?alt=media&token=47ad187d-cd3d-49d7-9793-1b090405ca50'
	];

	let scene: THREE.Scene;
	let camera: THREE.PerspectiveCamera;
	let renderer: THREE.WebGLRenderer;

	let clickableObjects: THREE.Object3D<THREE.Object3DEventMap>[] = [];

	onMount(async () => {
		initScene();
		addClickFunctionality();

		keySFX.forEach((sound) => {
			const audio = new Audio(sound);
			audio.load();
		});

		// const controls = new OrbitControls(camera, renderer.domElement);

		let gadget: THREE.Group<THREE.Object3DEventMap> | null = null;
		loadGLTF('/models/music-gadget.gltf').then((mesh) => {
			gadget = mesh;
			if (gadget) {
				scene.add(gadget);
				gadget.rotation.set(0, 0, 0);
				gadget.castShadow = true;

				gadget.children.forEach((element) => {
					const index = gadget?.children.indexOf(element);
					element.castShadow = true;

					if (index) {
						if (index > 0 && index < 10) {
							clickableObjects.push(element);
						}
					}
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

		const pointLight = new THREE.PointLight(0x3025fc, 15);
		pointLight.position.set(5, 4, -3);
		pointLight.castShadow = true;
		scene.add(pointLight);

		pointLight.shadow.mapSize.width = 2048; // default
		pointLight.shadow.mapSize.height = 2048; // default
		pointLight.shadow.camera.near = 0.05; // default
		pointLight.shadow.camera.far = 500; // default
		pointLight.shadow.bias = -0.0001;

		const pointLight2 = new THREE.PointLight(0xfc25f8, 15);
		pointLight2.position.set(-6, 4, -2);
		pointLight2.castShadow = true;
		scene.add(pointLight2);

		pointLight2.shadow.mapSize.width = 2048; // default
		pointLight2.shadow.mapSize.height = 2048; // default
		pointLight2.shadow.camera.near = 0.05; // default
		pointLight2.shadow.camera.far = 500; // default
		pointLight2.shadow.bias = -0.0001;

		// animation
		function animation(time: number) {
			renderer.render(scene, camera);

			// controls.update();
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
		// renderer.shadowMap.type = THREE.PCFShadowMap;

		document.getElementById('threeContainer')?.appendChild(renderer.domElement);
	}

	function loadGLTF(url: string): Promise<THREE.Group<THREE.Object3DEventMap> | null> {
		const loader = new GLTFLoader();

		return new Promise((resolve, reject) => {
			loader.load(
				url,
				function (gltf) {
					gltf.scene.children.map((child) => {
						const mesh = child as THREE.Mesh;
						mesh.receiveShadow = true;
						if (Array.isArray(mesh.material)) {
							mesh.material.forEach((material) => {
								material.side = THREE.DoubleSide;
							});
						} else {
							mesh.material.side = THREE.DoubleSide;
						}
					});
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

	function addClickFunctionality() {
		const raycaster = new THREE.Raycaster();
		const mouse = new THREE.Vector2();
		raycaster.far = 1000;

		window.addEventListener('click', (event) => {
			mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
			mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

			raycaster.setFromCamera(mouse, camera);

			const intersects = raycaster.intersectObjects(clickableObjects);

			if (intersects.length > 0) {
				const clickableObject = intersects[0].object;
				console.log('You clicked on ' + clickableObject.name);

				const audio = new Audio(keySFX[Math.floor(Math.random() * keySFX.length)]);
				audio.play();

				const originalPos = clickableObject.position.y;

				gsap.to(clickableObject.position, {
					duration: 0.01,
					y: 0.4,
					// ease: 'power1.inOut',

					onComplete: function () {
						gsap.to(clickableObject.position, {
							duration: 0.1,
							y: originalPos,
							ease: 'power1.inOut'
						});
					}
				});
			} else {
				console.log('Did not hit anything');
			}
		});
	}
</script>

<div id="threeContainer"></div>

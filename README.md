# Gand-Faad-Game
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Slow Roads - Ultimate Supercar Experience</title>
  <style>
    body { margin: 0; overflow: hidden; }
    canvas { display: block; }
    #info { position: absolute; top: 10px; left: 10px; color: white; font-family: monospace; background: rgba(0,0,0,0.5); padding: 10px; border-radius: 8px; }
    #select-car { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); color: white; font-family: monospace; text-align: center; font-size: 30px; }
    #select-car button { background: rgba(0, 0, 0, 0.7); border: none; color: white; padding: 10px 20px; font-size: 20px; margin: 10px; cursor: pointer; border-radius: 8px; }
    #select-car button:hover { background: rgba(0, 0, 0, 0.9); }
  </style>
</head>
<body>
  <div id="info">Press 1-5 to switch car: 1=Lambo 2=Bugatti 3=Koenigsegg 4=Mustang 5=BMW</div>
  <div id="select-car">
    <p>Select Your Car:</p>
    <button onclick="startGame(1)">Lamborghini</button>
    <button onclick="startGame(2)">Bugatti</button>
    <button onclick="startGame(3)">Koenigsegg</button>
    <button onclick="startGame(4)">Mustang</button>
    <button onclick="startGame(5)">BMW</button>
  </div>
  <audio id="bgm" src="https://cdn.pixabay.com/download/audio/2022/03/15/audio_3ab6fa3b1b.mp3?filename=calm-lofi-110094.mp3" autoplay loop></audio>
  <script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/simplex-noise@3.0.0/simplex-noise.min.js"></script>
  <script>
    let scene, camera, renderer, car, currentCarType = 1, speed = 0.5;
    let road = [], terrain = [], trees = [];
    const simplex = new SimplexNoise();
    const carColors = [0xffaa00, 0x00ccff, 0xcc0000, 0xffffff, 0x3333cc];
    let carModelUrls = [
      'https://example.com/lambo.glb',
      'https://example.com/bugatti.glb',
      'https://example.com/koenigsegg.glb',
      'https://example.com/mustang.glb',
      'https://example.com/bmw.glb'
    ];

    init();
    animate();

    function init() {
      scene = new THREE.Scene();
      scene.fog = new THREE.Fog(0xffcc99, 20, 100);
      scene.background = new THREE.Color(0xffcc99);

      camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
      camera.position.set(0, 5, -10);

      renderer = new THREE.WebGLRenderer({ antialias: true });
      renderer.setSize(window.innerWidth, window.innerHeight);
      document.body.appendChild(renderer.domElement);

      const light = new THREE.DirectionalLight(0xffffff, 1);
      light.position.set(10, 10, 10);
      scene.add(light);
      scene.add(new THREE.AmbientLight(0x404040));
    }

    function startGame(carType) {
      currentCarType = carType;
      spawnCar(currentCarType);
      generateInitialRoad();
      generateInitialTerrain();
      generateInitialTrees();
      document.getElementById('select-car').style.display = 'none';
    }

    function spawnCar(type) {
      if (car) scene.remove(car);
      const loader = new THREE.GLTFLoader();
      loader.load(carModelUrls[type - 1], function (gltf) {
        car = gltf.scene;
        car.scale.set(0.5, 0.5, 0.5);
        scene.add(car);
      });
    }

    function generateInitialRoad() {
      const roadMaterial = new THREE.MeshStandardMaterial({ color: 0x333333 });
      for (let i = 0; i < 100; i++) {
        const roadGeometry = new THREE.BoxGeometry(5, 0.1, 5);
        const roadPiece = new THREE.Mesh(roadGeometry, roadMaterial);
        roadPiece.position.set(0, -0.25, i * 5);
        scene.add(roadPiece);
        road.push(roadPiece);
      }
    }

    function generateInitialTerrain() {
      const terrainMaterial = new THREE.MeshStandardMaterial({ color: 0x228B22 });
      for (let i = 0; i < 100; i++) {
        for (let j = -6; j <= 6; j++) {
          if (Math.abs(j) < 3) continue;
          const height = getTerrainHeight(j, i);
          const terrainGeom = new THREE.BoxGeometry(1, height, 5);
          const terrainPiece = new THREE.Mesh(terrainGeom, terrainMaterial);
          terrainPiece.position.set(j, -height / 2, i * 5);
          scene.add(terrainPiece);
          terrain.push(terrainPiece);
        }
      }
    }

    function generateInitialTrees() {
      const treeMaterial = new THREE.MeshStandardMaterial({ color: 0x8B4513 });
      const leafMaterial = new THREE.MeshStandardMaterial({ color: 0x2e8b57 });
      for (let i = 0; i < 100; i++) {
        const z = i * 5;
        [-5, 5].forEach(x => {
          if (Math.random() > 0.6) {
            const trunk = new THREE.Mesh(new THREE.CylinderGeometry(0.1, 0.1, 1), treeMaterial);
            const leaves = new THREE.Mesh(new THREE.SphereGeometry(0.5), leafMaterial);
            trunk.position.set(x, 0.5, z);
            leaves.position.set(x, 1.5, z);
            scene.add(trunk);
            scene.add(leaves);
            trees.push(trunk, leaves);
          }
        });
      }
    }

    function getTerrainHeight(x, z) {
      return Math.abs(simplex.noise2D(x * 0.2, z * 0.1)) * 3 + 0.1;
    }

    function recycleEnvironment() {
      const lastZ = road[road.length - 1].position.z;

      if (car.position.z > road[10].position.z) {
        const removedRoad = road.shift();
        removedRoad.position.z = lastZ + 5;
        removedRoad.position.x = Math.sin((lastZ + 5) * 0.05) * 2;
        road.push(removedRoad);

        terrain.forEach(piece => {
          if (piece.position.z < car.position.z - 10) {
            piece.position.z = lastZ + 5;
            const height = getTerrainHeight(piece.position.x, (lastZ + 5) / 5);
            piece.scale.y = height;
            piece.position.y = -height / 2;
          }
        });

        trees.forEach(tree => {
          if (tree.position.z < car.position.z - 10) {
            tree.position.z = lastZ + 5 + Math.random() * 5;
            tree.position.x = Math.random() > 0.5 ? -5 : 5;
          }
        });
      }
    }

    function animate() {
      requestAnimationFrame(animate);
      car.position.z += speed;
      const targetX = Math.sin(car.position.z * 0.05) * 2;
      car.position.x = THREE.MathUtils.lerp(car.position.x, targetX, 0.05);

      // Car tilt
      car.rotation.z = THREE.MathUtils.lerp(car.rotation.z, (targetX - car.position.x) * 0.2, 0.1);

      // Camera follow
      camera.position.z = car.position.z - 10;
      camera.position.x = car.position.x;
      camera.lookAt(car.position);

      recycleEnvironment();
      renderer.render(scene, camera);
    }

    window.addEventListener('keydown', (e) => {
      if (e.key === 'ArrowLeft') car.position.x -= 0.3;
      if (e.key === 'ArrowRight') car.position.x += 0.3;
      if (e.key === 'ArrowUp') speed += 0.1;
      if (e.key === 'ArrowDown') speed = Math.max(0.1, speed - 0.1);
    });
  </script>
</body>
</html>

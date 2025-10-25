# tofa-3d-similasyon<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8">
<title>Tofaş Sürüş Simülatörü v1</title>
<style>
    body { margin: 0; overflow: hidden; }
    #info { position: absolute; top: 10px; left: 10px; color: white; font-family: Arial; background: rgba(0,0,0,0.5); padding:10px; border-radius:5px; }
</style>
</head>
<body>
<div id="info">
    W/S: İleri/Geri | ←/→: Direksiyon | Q/E: Vites | Z/X: Sinyal | H: Korna | L: Far | P: El freni
    <br>Vites: <span id="gear">N</span> | Hız: <span id="speed">0</span> km/h | Motor Sıcaklığı: <span id="temp">20</span>°C
</div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r150/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/cannon-es@0.20.0/dist/cannon-es.js"></script>

<script>
// Temel sahne
let scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb); // gökyüzü mavisi
let camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
let renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// Işıklar
let ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
scene.add(ambientLight);
let dirLight = new THREE.DirectionalLight(0xffffff, 0.6);
dirLight.position.set(10, 10, 10);
scene.add(dirLight);

// Zemin
let groundGeometry = new THREE.PlaneGeometry(500,500);
let groundMaterial = new THREE.MeshPhongMaterial({color:0x228B22});
let ground = new THREE.Mesh(groundGeometry, groundMaterial);
ground.rotation.x = -Math.PI/2;
scene.add(ground);

// Fizik dünyası
const world = new CANNON.World();
world.gravity.set(0,-9.82,0);

// Zemin fizik
let groundBody = new CANNON.Body({ mass:0 });
let groundShape = new CANNON.Plane();
groundBody.addShape(groundShape);
groundBody.quaternion.setFromEuler(-Math.PI/2,0,0);
world.addBody(groundBody);

// Araç
let carGeometry = new THREE.BoxGeometry(2,1,4);
let carMaterial = new THREE.MeshPhongMaterial({color:0xff0000});
let car = new THREE.Mesh(carGeometry, carMaterial);
scene.add(car);

let carBody = new CANNON.Body({ mass:1200 });
let carShape = new CANNON.Box(new CANNON.Vec3(1,0.5,2));
carBody.addShape(carShape);
carBody.position.set(0,2,0);
world.addBody(carBody);

// Kamera içten
camera.position.set(0,1.5,-0.5);
camera.lookAt(new THREE.Vector3(0,1.5,5));
car.add(camera);

// Kontroller
let keys = {};
document.addEventListener('keydown', (e)=>{ keys[e.key.toLowerCase()] = true; });
document.addEventListener('keyup', (e)=>{ keys[e.key.toLowerCase()] = false; });

// Vites ve motor
let gear = 0;
let speed = 0;
let motorTemp = 20;

// Ana döngü
function animate(){
    requestAnimationFrame(animate);

    // Araç kontrolleri
    let forward = 0;
    if(keys['w']) forward = 200;
    if(keys['s']) forward = -100;
    carBody.applyLocalForce(new CANNON.Vec3(0,0,forward), new CANNON.Vec3(0,0,0));

    // Direksiyon
    if(keys['arrowleft']) carBody.quaternion.setFromEuler(0, carBody.quaternion.toEuler().y + 0.03,0);
    if(keys['arrowright']) carBody.quaternion.setFromEuler(0, carBody.quaternion.toEuler().y - 0.03,0);

    // Vites
    if(keys['q']) { gear = Math.max(0, gear-1); keys['q']=false; }
    if(keys['e']) { gear = Math.min(5, gear+1); keys['e']=false; }

    // Hız ve motor sıcaklığı
    speed = carBody.velocity.length()*3.6; // km/h
    motorTemp += Math.min(speed*0.001, 0.05);

    // Göstergeleri güncelle
    document.getElementById('gear').innerText = gear==0?'N':gear;
    document.getElementById('speed').innerText = Math.floor(speed);
    document.getElementById('temp').innerText = motorTemp.toFixed(1);

    world.step(1/60);
    car.position.copy(carBody.position);
    car.quaternion.copy(carBody.quaternion);

    renderer.render(scene, camera);
}
animate();
</script>
</body>
</html>

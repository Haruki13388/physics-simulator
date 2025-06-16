# cd physics-simulator             # Go to your project folder
git init                          # Initialize git
git add .                         # Add all files
git commit -m "Initial commit"    # Commit
git branch -M main                # Set main branch
git remote add origin https://github.com/YOUR-USERNAME/physics-simulator.git
git push -u origin main           # Push code to GitHub
import { useRef, useEffect } from 'react';
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';

export default function UniverseCanvas() {
  const mountRef = useRef(null);

  useEffect(() => {
    const width = mountRef.current.clientWidth;
    const height = mountRef.current.clientHeight;
    const scene = new THREE.Scene();
    scene.background = new THREE.Color('black');

    const camera = new THREE.PerspectiveCamera(60, width / height, 0.1, 1000);
    camera.position.set(0, 40, 100);

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(width, height);
    mountRef.current.appendChild(renderer.domElement);

    const controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;

    const ambientLight = new THREE.AmbientLight(0xffffff, 0.3);
    scene.add(ambientLight);
    const pointLight = new THREE.PointLight(0xffffff, 1.5, 0);
    pointLight.position.set(0, 0, 0);
    scene.add(pointLight);

    // Sun (Star)
    const starGeometry = new THREE.SphereGeometry(8, 32, 32);
    const starMaterial = new THREE.MeshBasicMaterial({ color: 0xffee88 });
    const star = new THREE.Mesh(starGeometry, starMaterial);
    scene.add(star);

    // Planets
    const planets = [];
    const planetData = [
      { color: 0x3366ff, radius: 2, distance: 20, orbitSpeed: 0.008, theta: 0 },
      { color: 0x55ff55, radius: 3, distance: 32, orbitSpeed: 0.005, theta: Math.PI / 2 },
      { color: 0xff5555, radius: 1.5, distance: 45, orbitSpeed: 0.012, theta: Math.PI }
    ];
    planetData.forEach((data) => {
      const geometry = new THREE.SphereGeometry(data.radius, 32, 32);
      const material = new THREE.MeshStandardMaterial({
        color: data.color, roughness: 0.6, metalness: 0.2
      });
      const mesh = new THREE.Mesh(geometry, material);
      mesh.userData = data;
      scene.add(mesh);
      planets.push(mesh);
    });

    // Starfield
    const starCount = 400;
    const starGeometry2 = new THREE.BufferGeometry();
    const starVertices = [];
    for (let i = 0; i < starCount; i++) {
      const x = THREE.MathUtils.randFloatSpread(400);
      const y = THREE.MathUtils.randFloatSpread(400);
      const z = THREE.MathUtils.randFloatSpread(400);
      starVertices.push(x, y, z);
    }
    starGeometry2.setAttribute('position', new THREE.Float32BufferAttribute(starVertices, 3));
    const starMaterial2 = new THREE.PointsMaterial({ color: 0xffffff, size: 1.2 });
    const stars = new THREE.Points(starGeometry2, starMaterial2);
    scene.add(stars);

    // Animation
    let animationId;
    function animate() {
      planets.forEach((planet) => {
        planet.userData.theta += planet.userData.orbitSpeed;
        planet.position.x = Math.cos(planet.userData.theta) * planet.userData.distance;
        planet.position.z = Math.sin(planet.userData.theta) * planet.userData.distance;
        planet.rotation.y += 0.01;
      });
      controls.update();
      renderer.render(scene, camera);
      animationId = requestAnimationFrame(animate);
    }
    animate();

    // Cleanup
    return () => {
      cancelAnimationFrame(animationId);
      renderer.dispose();
      mountRef.current.removeChild(renderer.domElement);
      scene.traverse((obj) => {
        if (obj.geometry) obj.geometry.dispose();
        if (obj.material) {
          if (Array.isArray(obj.material)) obj.material.forEach((m) => m.dispose());
          else obj.material.dispose();
        }
      });
    };
  }, []);

  return (
    <div ref={mountRef} style={{
      width: "100%", height: "600px", borderRadius: '8px', overflow: 'hidden'
    }} />
  );
}

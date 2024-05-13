<template>
  <div ref="container"></div>
</template>

<script>
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

export default {
  name: 'GlobeView',
  data() {
    return {
      radioStations: [],
    };
  },
  mounted() {
    this.fetchRadioStations();
  },
  methods: {
    fetchRadioStations() {
      fetch('https://nl1.api.radio-browser.info/json/stations/search?limit=1000&hidebroken=true&order=clickcount&reverse=true')
      .then(response => response.json())
      .then(data => {
          this.radioStations = data;
          this.createGlobe();
        })
      .catch(error => {
          console.error('Error fetching radio stations:', error);
        });
    },
    createGlobe() {
      const scene = new THREE.Scene();
      const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
      const renderer = new THREE.WebGLRenderer();
      renderer.setSize(window.innerWidth, window.innerHeight);
      this.$refs.container.appendChild(renderer.domElement);

      const geometry = new THREE.SphereGeometry(5, 32, 32);
      const texture = new THREE.TextureLoader().load(require('@/assets/earth.jpg'));
      const material = new THREE.MeshBasicMaterial({ map: texture });
      const globe = new THREE.Mesh(geometry, material);
      scene.add(globe);

      camera.position.set(0, 0, 15);
      camera.lookAt(0, 0, 0);

      const controls = new OrbitControls(camera, renderer.domElement);
      controls.update();

      this.radioStations.forEach(station => {
        const latitude = parseFloat(station.geo_lat);
        const longitude = parseFloat(station.geo_long);
        this.addMarker(latitude, longitude, scene);
      });

      const animate = () => {
        requestAnimationFrame(animate);
        controls.update();
        renderer.render(scene, camera);
      };
      animate();
    },
    addMarker(latitude, longitude, scene) {
      if (!isNaN(latitude) &&!isNaN(longitude)) {
        const phi = (90 - latitude) * (Math.PI / 180);
        const theta = (longitude + 180) * (Math.PI / 180);
        const x = -5 * Math.sin(phi) * Math.cos(theta);
        const y = 5 * Math.cos(phi);
        const z = 5 * Math.sin(phi) * Math.sin(theta);

        const geometry = new THREE.SphereGeometry(0.1, 8, 8);
        const material = new THREE.MeshBasicMaterial({ color: 0xff0000 });
        const marker = new THREE.Mesh(geometry, material);
        marker.position.set(x, y, z);
        scene.add(marker);
      } else {
        console.error('Latitude or longitude is not a number:', latitude, longitude);
      }
    },
  },
};
</script>

<style scoped>
#container {
  width: 100%;
  height: 100%;
}
</style>

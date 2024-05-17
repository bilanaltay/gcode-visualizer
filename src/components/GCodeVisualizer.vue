<template>
  <div>
    <input type="file" @change="handleFile" />
    <div id="container" ref="container"></div>
  </div>
</template>

<script>
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';

export default {
  methods: {
    handleFile(event) {
      const file = event.target.files[0];
      const reader = new FileReader();

      reader.onload = (e) => {
        const gcode = e.target.result;
        this.visualizeGCode(gcode);
      };

      reader.readAsText(file);
    },
    visualizeGCode(gcode) {
      const container = this.$refs.container;
      container.innerHTML = '';

      const scene = new THREE.Scene();
      const camera = new THREE.PerspectiveCamera(75, container.clientWidth / container.clientHeight, 0.1, 1000);
      const renderer = new THREE.WebGLRenderer();
      renderer.setSize(container.clientWidth, container.clientHeight);
      container.appendChild(renderer.domElement);

      const controls = new OrbitControls(camera, renderer.domElement);

      const ambientLight = new THREE.AmbientLight(0x404040);
      scene.add(ambientLight);
      const directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
      scene.add(directionalLight);

      const lines = gcode.split('\n');
      const material = new THREE.LineBasicMaterial({ color: 0x0000ff });
      const points = [];
      let x = 0, y = 0, z = 0;

      lines.forEach(line => {
        const parsed = this.parseGCodeLine(line);

        if (parsed.cmd) {
          switch (parsed.cmd) {
            case 'G0':
            case 'G1':
              ({ x, y, z } = this.updateCoords(parsed.words, x, y, z));
              points.push(new THREE.Vector3(x, y, z));
              break;
            case 'G2':
            case 'G3':
              this.drawArc3D(parsed, scene, x, y, z);
              ({ x, y, z } = this.updateCoords(parsed.words, x, y, z));
              break;
            default:
              break;
          }
        }
      });

      const geometry = new THREE.BufferGeometry().setFromPoints(points);
      const line = new THREE.Line(geometry, material);
      scene.add(line);

      camera.position.z = 50;

      const animate = () => {
        requestAnimationFrame(animate);
        controls.update();
        renderer.render(scene, camera);
      };

      animate();
    },
    parseGCodeLine(line) {
      const words = line.match(/[A-Z][^A-Z]*/g) || [];
      const parsed = { cmd: null, words: [] };

      words.forEach(word => {
        const letter = word[0];
        const value = parseFloat(word.slice(1));

        if (letter === 'G' || letter === 'M') {
          parsed.cmd = `${letter}${value}`;
        } else {
          parsed.words.push({ letter, value });
        }
      });

      return parsed;
    },
    updateCoords(words, currentX, currentY, currentZ) {
      const x = this.getCoord(words, 'X', currentX);
      const y = this.getCoord(words, 'Y', currentY);
      const z = this.getCoord(words, 'Z', currentZ);
      return { x, y, z };
    },
    getCoord(words, axis, currentValue) {
      const word = words.find(word => word.letter === axis);
      return word ? parseFloat(word.value) : currentValue;
    },
    drawArc3D(parsed, scene, startX, startY, startZ) {
      const i = this.getCoord(parsed.words, 'I', 0);
      const j = this.getCoord(parsed.words, 'J', 0);
      const endX = this.getCoord(parsed.words, 'X', startX);
      const endY = this.getCoord(parsed.words, 'Y', startY);
      const centerX = startX + i;
      const centerY = startY + j;

      const startAngle = Math.atan2(startY - centerY, startX - centerX);
      const endAngle = Math.atan2(endY - centerY, endX - centerX);
      const radius = Math.sqrt(i * i + j * j);

      const points = [];
      const numSegments = 100;
      const angleStep = (endAngle - startAngle) / numSegments;

      for (let k = 0; k <= numSegments; k++) {
        const angle = startAngle + angleStep * k;
        const x = centerX + radius * Math.cos(angle);
        const y = centerY + radius * Math.sin(angle);
        points.push(new THREE.Vector3(x, y, startZ));
      }

      const geometry = new THREE.BufferGeometry().setFromPoints(points);
      const material = new THREE.LineBasicMaterial({ color: 0xff0000 });
      const arcLine = new THREE.Line(geometry, material);
      scene.add(arcLine);
    }
  }
};
</script>

<style scoped>
#container {
  width: 100%;
  height: 600px;
}
</style>

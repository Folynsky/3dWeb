<template>
    <div>
      <div id="info">
        <a href="https://threejs.org" target="_blank" rel="noopener">three.js</a> - SDF to Mesh -
        <br />
        a wrapper of
        <a href="https://www.npmjs.com/package/isosurface" target="_blank">Mikola Lysenko's Isosurface</a>
        <br />
        Mandelbrot by
        <a href="https://www.shadertoy.com/view/MdXSWn" target="_blank">EvilRyu</a>
      </div>
      <div ref="threeContainer"></div>
    </div>
  </template>
  
  <script>
  import * as THREE from 'three';
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
  import { SDFGeometryGenerator } from 'three/examples/jsm/geometries/SDFGeometryGenerator';
  import Stats from 'three/examples/jsm/libs/stats.module';
  import { GUI } from 'lil-gui';
  
  export default {
    data() {
      return {
        renderer: null,
        stats: null,
        meshFromSDF: null,
        scene: null,
        camera: null,
        clock: null,
        controls: null,
        settings: {
          res: 4,
          bounds: 1,
          autoRotate: true,
          wireframe: true,
          material: 'depth',
          vertexCount: '0',
        },
        shader: `
          float dist(vec3 p) {
            p.xyz = p.xzy;
            p *= 1.2;
            vec3 z = p;
            vec3 dz = vec3(0.0);
            float power = 8.0;
            float r, theta, phi;
            float dr = 1.0;
            
            float t0 = 1.0;
            for(int i = 0; i < 7; ++i) {
              r = length(z);
              if(r > 2.0) continue;
              theta = atan(z.y / z.x);
              #ifdef phase_shift_on
              phi = asin(z.z / r);
              #else
              phi = asin(z.z / r);
              #endif
              
              dr = pow(r, power - 1.0) * dr * power + 1.0;
            
              r = pow(r, power);
              theta = theta * power;
              phi = phi * power;
              
              z = r * vec3(cos(theta)*cos(phi), sin(theta)*cos(phi), sin(phi)) + p;
              
              t0 = min(t0, r);
            }
  
            return 0.5 * log(r) * r / dr;
          }
        `,
      };
    },
    mounted() {
      this.init();
    },
    methods: {
      init() {
        const w = this.$refs.threeContainer.clientWidth;
        const h = this.$refs.threeContainer.clientHeight;
  
        this.camera = new THREE.OrthographicCamera(w / -2, w / 2, h / 2, h / -2, 0.01, 1600);
        this.camera.position.z = 1100;
  
        this.scene = new THREE.Scene();
        this.clock = new THREE.Clock();
  
        this.renderer = new THREE.WebGLRenderer({ antialias: true });
        this.renderer.setPixelRatio(window.devicePixelRatio);
        this.renderer.setSize(w, h);
        this.renderer.setAnimationLoop(this.animate);
        this.$refs.threeContainer.appendChild(this.renderer.domElement);
  
        this.stats = new Stats();
        this.$refs.threeContainer.appendChild(this.stats.dom);
  
        this.controls = new OrbitControls(this.camera, this.renderer.domElement);
        this.controls.enableDamping = true;
  
        window.addEventListener('resize', this.onWindowResize);
  
        const panel = new GUI();
        panel.add(this.settings, 'res', 1, 6, 1).name('Res').onFinishChange(this.compile);
        panel.add(this.settings, 'bounds', 1, 10, 1).name('Bounds').onFinishChange(this.compile);
        panel.add(this.settings, 'material', ['depth', 'normal']).name('Material').onChange(this.setMaterial);
        panel.add(this.settings, 'wireframe').name('Wireframe').onChange(this.setMaterial);
        panel.add(this.settings, 'autoRotate').name('Auto Rotate');
        panel.add(this.settings, 'vertexCount').name('Vertex count').listen().disable();
  
        this.compile();
      },
      compile() {
        const generator = new SDFGeometryGenerator(this.renderer);
        const geometry = generator.generate(Math.pow(2, this.settings.res + 2), this.shader, this.settings.bounds);
        geometry.computeVertexNormals();
  
        if (this.meshFromSDF) {
          this.meshFromSDF.geometry.dispose();
          this.meshFromSDF.geometry = geometry;
        } else {
          this.meshFromSDF = new THREE.Mesh(geometry, new THREE.MeshBasicMaterial());
          this.scene.add(this.meshFromSDF);
  
          const scale = Math.min(window.innerWidth, window.innerHeight) / 2 * 0.66;
          this.meshFromSDF.scale.set(scale, scale, scale);
  
          this.setMaterial();
        }
  
        this.settings.vertexCount = geometry.attributes.position.count;
      },
      setMaterial() {
        this.meshFromSDF.material.dispose();
  
        if (this.settings.material === 'depth') {
          this.meshFromSDF.material = new THREE.MeshDepthMaterial();
        } else if (this.settings.material === 'normal') {
          this.meshFromSDF.material = new THREE.MeshNormalMaterial();
        }
  
        this.meshFromSDF.material.wireframe = this.settings.wireframe;
      },
      onWindowResize() {
        const w = window.innerWidth;
        const h = window.innerHeight;
  
        this.renderer.setSize(w, h);
  
        this.camera.left = w / -2;
        this.camera.right = w / 2;
        this.camera.top = h / 2;
        this.camera.bottom = h / -2;
  
        this.camera.updateProjectionMatrix();
      },
      render() {
        this.renderer.render(this.scene, this.camera);
      },
      animate() {
        this.controls.update();
  
        if (this.settings.autoRotate) {
          this.meshFromSDF.rotation.y += Math.PI * 0.05 * this.clock.getDelta();
        }
  
        this.render();
        this.stats.update();
      },
    },
  };
  </script>
  
  <style scoped>
  #info {
    position: absolute;
    top: 10px;
    left: 10px;
    color: white;
    font-family: sans-serif;
  }
  </style>
  
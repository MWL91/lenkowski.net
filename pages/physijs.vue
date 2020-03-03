<template>
    <div>
    </div>
</template>

<script>
import Logo from "~/components/Logo.vue";
import * as THREE from "three";
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';

export default {
    components: {
        Logo
    },

    data() {
        return {
            scene: new THREE.Scene(),
            camera: new THREE.PerspectiveCamera(
                70,
                window.innerWidth / window.innerHeight,
                1,
                1000
            ),
            renderer: new THREE.WebGLRenderer({ antialias: true }),
            meshs: {},
            car: null,
            animateRequest: null,
            pos: 0
        }
    },

    methods: {
        initScene() {
            this.camera.position.z = 5;

            // let cube = this.addCube();
            // cube.position.x = 2;
            // this.animateRotation(cube);

            // let cube2 = this.addCube(2, 1, 1);
            // this.animateRotation(cube2, 0.3, 0.3);

            // this.addLine();

            // let txt = this.addText();
            // // txt.position.x = -5;
            // txt.position.z = 2;
            // this.animateRotation(txt);

            this.addJeep();

            this.renderer.setSize(window.innerWidth, window.innerHeight);

            document.body.appendChild(this.renderer.domElement);
        },

        addCube(width = 0.2, height = 0.2, depth = 0.2) {
            let geometry = new THREE.BoxGeometry(width, height, depth);
            let material = new THREE.MeshNormalMaterial();
            let mesh = new THREE.Mesh(geometry, material);

            this.scene.add(mesh);
            return mesh;
        },

        addLine() {
            let material = new THREE.LineBasicMaterial( { color: 0xff0000 } );

            let points = [];
            points.push( new THREE.Vector3( - 3, 0, 0 ) );
            points.push( new THREE.Vector3( 0, 3, 0 ) );
            points.push( new THREE.Vector3( 3, 0, 0 ) );

            let geometry = new THREE.BufferGeometry().setFromPoints( points );

            let line = new THREE.Line( geometry, material );

            this.scene.add(line);
            return line;
        },

        addText() {
            let material = new THREE.MeshNormalMaterial();
            let loader = new THREE.FontLoader();
            
            let font = new THREE.Font(require('~/static/Lato_Regular.json'));
            let geometry = new THREE.TextGeometry( 'Martusia <3', {
                font: font,
                size: 1,
                height: 0.1,
            } );

            let mesh = new THREE.Mesh(geometry, material);
            this.scene.add(mesh);
            return mesh;

        },

        addObject() {
            var loader = new GLTFLoader();
            let self = this;

            let light = new THREE.AmbientLight( 0xFFFFFF ); // soft white light
            this.scene.add( light );

            loader.load( 'glb/duck.glb', function ( gltf ) {
                self.scene.add( gltf.scene );
                self.animateRotation( gltf.scene );
            }, undefined, function ( error ) {
                console.error( error );
            } );
        },

        addJeep() {
            var loader = new GLTFLoader();
            let self = this;

            // let light = new THREE.AmbientLight( 0x00FFFF ); // soft white light
            let light = new THREE.DirectionalLight( 0xffffff, 1 );
            this.scene.add( light );

            loader.load( 'glb/duck.glb', function ( gltf ) {
                gltf.scene.position.z = -5;
                gltf.scene.position.y = -2;
                self.scene.add( gltf.scene );
                
                self.setCar(gltf.scene);
                
                self.renderer.render( self.scene, self.camera );
            }, undefined, function ( error ) {
                console.error( error );
            } );
        },

        setCar(mesh) {
            this.car = mesh;
        },

        animateCamera() {
            requestAnimationFrame( this.animateCamera );

            // this.camera.fov += 1;


            this.camera.position.z += 0.01;
            this.camera.rotation.y += 0.01;

            // this.camera.position.x = current % 10;
            // this.camera.rotation.y = current % 10;
            // this.camera.position.y = current % 10;
            // this.camera.position.z = current % 10;
            
            this.camera.updateProjectionMatrix();
 
            this.renderer.render( this.scene, this.camera );
        },

        animateRotation(mesh, rotationX = 0.02, rotationY = 0.02) {
            const animateRotation = this.animateRotation;
            requestAnimationFrame( function() {
                animateRotation(mesh);
            } );

            mesh.rotation.x += rotationX;
            mesh.rotation.y += rotationY;
        
            this.renderer.render( this.scene, this.camera );
        },

        animateForward(mesh, forward = 0.05, radius = 5) {
            const animateForward = this.animateForward;
            this.animateRequest = requestAnimationFrame( function() {
                animateForward(mesh);
            } );

            this.setCarStep(mesh, forward, radius);
        },

        animateBackward(mesh, forward = -0.05, radius = 5) {
            const animateBackward = this.animateBackward;
            this.animateRequest = requestAnimationFrame( function() {
                animateBackward(mesh);
            } );

            this.setCarStep(mesh, forward, radius);
        },

        setCarStep(mesh, forward, length) {
            this.pos += forward;

            mesh.rotation.y = -1 * ((Math.PI/180) * this.pos/(Math.PI/180) % 360);
            mesh.position.z = (Math.sin(this.pos) * length);
            mesh.position.x = (Math.cos(this.pos) * length);

        
            this.renderer.render( this.scene, this.camera );
        }
    },

    mounted() {
        this.initScene();
        // this.animateCamera();

        let self = this;

        document.body.onkeydown = function(e) {
            cancelAnimationFrame(self.animateRequest);

            switch(e.key) {
                case 'w':
                    self.animateForward(self.car);
                break;
                case 's':
                    self.animateBackward(self.car);
                break;
            }
        };

        document.body.onkeyup = function(e) {
            cancelAnimationFrame(self.animateRequest);
        };
    }
};
</script>

<style>
/* Sample `apply` at-rules with Tailwind CSS
.container {
  @apply min-h-screen flex justify-center items-center text-center mx-auto;
}
*/
.container {
    margin: 0 auto;
    min-height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    text-align: center;
}

.title {
    font-family: "Quicksand", "Source Sans Pro", -apple-system,
        BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial,
        sans-serif;
    display: block;
    font-weight: 300;
    font-size: 100px;
    color: #35495e;
    letter-spacing: 1px;
}

.subtitle {
    font-weight: 300;
    font-size: 42px;
    color: #526488;
    word-spacing: 5px;
    padding-bottom: 15px;
}

.links {
    padding-top: 15px;
}


#info {
	position: absolute;
	top: 10px;
	width: 100%;
	text-align: center;
	z-index: 100;
	display:block;
}
</style>

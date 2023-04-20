<template>
    <div class="box">
        <div v-if="loading" class="cover">
            <div class="progress">{{ progress }}</div>
        </div>
        <div class="navbar" @click="change()"></div>
        <div class="play" @click="play()"></div>
        <div id="container">
        </div>
    </div>
</template>

<script>
import * as THREE from "three"; //引入Threejs
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader"; //模型加载器
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js"; //控制器


export default {
    name: 'ImportTemplate',
    data() {
        return {
            scene: null,
            light: null,
            camera: null,
            renderer: null,
            controls: null,
            mesh: null,
            group: null,
            progress: null,
            loading: true,
            animations: null,
            mixer: null,
            action: null,
            clock: new THREE.Clock(),
            id: 1,
        }
    },
    mounted() {
        this.init()
        this.loadGltf()
    },

    methods: {
        init() {
            let container = document.getElementById("container");//显示3D模型的容器
            this.scene = new THREE.Scene();
            this.light = new THREE.AmbientLight(0x888888)
            this.scene.add(this.light); //环境光
            const directionalLight1 = new THREE.DirectionalLight(0xffffff, 1);
            directionalLight1.position.set(100, 0, 0);
            this.scene.add(directionalLight1); //环境光
            const directionalLight2 = new THREE.DirectionalLight(0xffffff, 1);
            directionalLight2.position.set(0, 100, 0);
            this.scene.add(directionalLight2); //环境光
            const directionalLight3 = new THREE.DirectionalLight(0xffffff, 1);
            directionalLight3.position.set(0, 0, 100);
            this.scene.add(directionalLight3); //环境光
            const directionalLight4 = new THREE.DirectionalLight(0xffffff, 1);
            directionalLight4.position.set(-100, 0, 0);
            this.scene.add(directionalLight4); //环境光
            const directionalLight5 = new THREE.DirectionalLight(0xffffff, 1);
            directionalLight5.position.set(0, -100, 0);
            this.scene.add(directionalLight5); //环境光
            const directionalLight6 = new THREE.DirectionalLight(0xffffff, 1);
            directionalLight6.position.set(0, 0, -100);
            this.scene.add(directionalLight6); //环境光

            /**
             * 相机设置
             */

            this.camera = new THREE.PerspectiveCamera(
                45,
                container.clientWidth / container.clientHeight,
                0.01,
                10
            );
            this.camera.position.set(1, 1, 1);

            const axesHelper = new THREE.AxesHelper(5);
            this.scene.add(axesHelper);
            /**
             * 创建渲染器对象
             */
            this.renderer = new THREE.WebGLRenderer({ alpha: true });
            this.renderer.setSize(container.clientWidth, container.clientHeight);
            container.appendChild(this.renderer.domElement);
            this.controls = new OrbitControls(this.camera, this.renderer.domElement)

        },
        async loadGltf() {
            // 创建LoadingManager对象
            this.scene.remove(this.mesh)
            const manager = new THREE.LoadingManager();
            manager.onProgress = (item, loaded, total) => {
                this.progress = Math.round((loaded / total) * 100);
                if(this.progress === 100) {
                    console.log(this.progress);
                    this.loading = false;
                }
            };
            await this.loadModel(manager, this.id).then((model) => {
                this.mesh = model;
                this.scene.add(this.mesh); // 将模型引入three
                // 创建动画混合器
                this.mixer = new THREE.AnimationMixer(model)

                // 获取第一个动画
                let animation = this.animations[0]
                if(animation) {
                    console.log(animation, '111');

                    // 创建动画剪辑
                    let clip = THREE.AnimationClip.findByName(this.animations, animation.name)

                    // 创建动画动作
                    this.action = this.mixer.clipAction(clip)
                }
                this.animate();
            })
        },
        loadModel(manager, id) {
            return new Promise(resolve => {
                var loader = new GLTFLoader(manager);
                loader.load(`files/${ id }/model.glb`, (gltf) => {
                    let model = gltf.scene;
                    this.animations = gltf.animations
                    model.scale.set(0.3, 0.3, 0.3);//设置大小比例
                    model.position.set(0, 0, 0);//设置位置
                    resolve(model)
                })
            });
        },
        change() {
            this.mixer = null;
            this.id++;
            if(this.id > 2) {
                this.id = 1;
            }
            this.loadGltf()
        },
        play() {
            this.action.play() // 播放动画
        },
        animate() {
            if(this.mesh) {
                requestAnimationFrame(this.animate);
                // this.mesh.rotation.y += 0.004;//绕Y轴旋转0.004°
                this.controls.update()
                if(this.mixer) {
                    const deltaTime = this.clock.getDelta()
                    this.mixer.update(deltaTime) // 更新混合器
                }
                this.renderer.render(this.scene, this.camera);
            }
        },
    }
}
</script>

<style scoped>
.box {
    width: 100vw;
    height: 100vh;
}

.cover {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: beige;
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 999;
}

.progress {
    font-size: 20px;
    color: red;
}

.navbar {
    position: fixed;
    top: 10px;
    right: 10px;
    background-color: aqua;
    width: 50px;
    height: 50px;
    border-radius: 50%;
    z-index: 99;
}

.play {
    position: fixed;
    top: 10px;
    right: 70px;
    background-color: yellow;
    width: 50px;
    height: 50px;
    border-radius: 50%;
    z-index: 99;
}

#container {
    width: 100%;
    height: 100%;
    z-index: 10;
}
</style>


<template>
    <div>
        <h1>测试</h1>
        <div id="container" style="width: 650px;height: 650px;background-color:#f2f2f2;"></div>
    </div>
</template>

<script setup>
import {onMounted} from 'vue';
import * as THREE from 'three';
import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls.js';
//导入GLTF模块，模型解析器,根据文件格式来定
import {GLTFLoader} from 'three/examples/jsm/loaders/GLTFLoader';
import {DRACOLoader} from 'three/addons';
// gui
// import { GUI } from 'three/addons/libs/lil-gui.module.min.js';
// const gui = new GUI();
onMounted(() => {
    const container = document.getElementById('container');

    const {clientWidth, clientHeight} = container;

    // 创建场景
    const scene = new THREE.Scene();
    // const texture = new THREE.TextureLoader().load(require('@/assets/image/view-4.png'))
    // const texture = new THREE.TextureLoader().load(require('@/assets/image/model-bg-3.jpg'))
    // texture.mapping = THREE.EquirectangularReflectionMapping
    // scene.background = texture
    // scene.environment = texture
    // scene.backgroundIntensity = 0
    // scene.backgroundBlurriness = 1
    // texture.dispose()

    // 创建相机
    const camera = new THREE.PerspectiveCamera(50, clientWidth / clientHeight, 1, 2000);
    camera.position.set(0, 0, 5);

    // 创建渲染器
    const renderer = new THREE.WebGLRenderer({antialias:true, alpha:true, preserveDrawingBuffer:true}); //设置抗锯齿
    //设置屏幕像素比
    renderer.setPixelRatio(window.devicePixelRatio);
    //渲染的尺寸大小
    renderer.setSize(clientWidth, clientHeight);
    //色调映射
    renderer.toneMapping = THREE.ReinhardToneMapping;
    renderer.autoClear = true;
    renderer.outputColorSpace = THREE.SRGBColorSpace;
    //曝光
    // renderer.toneMappingExposure = 2;
    // renderer.shadowMap.enabled = true;
    // renderer.shadowMap.type = THREE.PCFSoftShadowMap;
    container.appendChild(renderer.domElement);

    // 创建环境光
    // const light ={
    //     color: '#fff',
    //     // intensity: 0.8,
    //     intensity: 1.1,
    // }
    // const ambientLight = new THREE.AmbientLight('#fff', light.intensity)
    //  ambientLight.visible = true
    // scene.add(ambientLight)
    // gui.add(ambientLight, 'intensity', 0, 5).name('环境光强度').onChange((v) => {
    //     ambientLight.intensity = v
    // })
    // 灯光 - 环境光
    const light = new THREE.AmbientLight(0xffffff, 0.67);
    scene.add(light);

    // 灯光 - 半球光
    const light2 = new THREE.HemisphereLight(0xffffff, 0x0444444, 10);
    light2.position.set(0, 20, 0);
    scene.add(light2);

    // 创建立方体
    // const geometry = new THREE.BoxGeometry();
    // const material = new THREE.MeshBasicMaterial({color:0x00ff00});
    // const cube = new THREE.Mesh(geometry, material);
    // scene.add(cube);

    // 导入模型
    const fileLoaderMap = {
        'glb':new GLTFLoader(),
        // 'fbx': new FBXLoader(),
        'gltf':new GLTFLoader(),
        // 'obj': new OBJLoader(),
    };
    loadModel({
        filePath:'/3D-2426-XL/2426-XL.glb',
        fileType:'glb',
        // map: this.modelMaterialList
    });

    function loadModel({filePath, fileType, /*map*/}) {
        new Promise((resolve, reject) => {
            const dracoLoader = new DRACOLoader();
            // 设置public下的解码路径，注意最后面的/
            dracoLoader.setDecoderPath('/draco/');
            //使用兼容性强的draco_decoder.js解码器
            dracoLoader.setDecoderConfig({type:'js'});
            dracoLoader.preload();
            const loader = fileLoaderMap[fileType];
            loader.setDRACOLoader(dracoLoader);
            loader.load(filePath, (result) => {
                const model = result.scene;
                const skeletonHelper = new THREE.SkeletonHelper(result.scene);
                // console.log('result',result);
                switch (fileType) {
                    case 'glb':
                        // this.model = result.scene;
                        // this.skeletonHelper = new THREE.SkeletonHelper(result.scene);
                        break;
                    case 'fbx':
                        // this.model = result;
                        // this.skeletonHelper = new THREE.SkeletonHelper(result);
                        break;
                    case 'gltf':
                        // this.model = result.scene;
                        // this.skeletonHelper = new THREE.SkeletonHelper(result.scene);
                        break;
                    case 'obj':
                        // this.model = result;
                        // this.skeletonHelper = new THREE.SkeletonHelper(result);
                        break;
                    default:
                        break;
                }
                // this.getModelMeaterialList(map);
                // this.modelAnimation = result.animations || [];
                // this.setModelPositionSize();
                // this.skeletonHelper.visible = false;
                // scene.add(this.skeletonHelper);
                // this.glowMaterialList = this.modelMaterialList.map(v => v.name);
                scene.add(skeletonHelper);
                scene.add(model);
                resolve(true);
            }, () => {

            }, (err) => {
                // ElMessage.error('文件错误');
                alert('文件错误');
                console.log(err);
                reject();
            });
        });
    }


    // 创建控制器
    const controls = new OrbitControls(camera, renderer.domElement);
    controls.enablePan = false; // 禁止平移
    controls.enableDamping = true; // 阻尼
    controls.target.set(0, 0, 0);
    controls.update();

    // 动画
    const animate = function () {
        controls.update();
        requestAnimationFrame(animate);
        renderer.render(scene, camera);
    };

    animate();
});
</script>

<style>
* {
    margin: 0;
    padding: 0;
}
</style>

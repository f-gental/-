<template>
    <div class="loading" ref="loading">
    </div>
    <TresCanvas window-size clear-color="black" preset="realistic" :tone-mapping-exposure="1" useLegacyLights shadows
        alpha ref="canvas">
        <TresPerspectiveCamera ref="camera" :position="[positions.positionx, positions.positiony, positions.positionz]"
            :look-at="[0, 0, 3]" />
        <!-- <OrbitControls /> -->
        <TresAxesHelper :args="[12]" />
        <!-- <Sky /> -->
        <Plane :args="[22, 22]" ref="plane">
            <TresMeshToonMaterial ref="MeshToon" />
        </Plane>
        :size-attenuation="true" />

        <TresMesh :position="[2, 3, 3]" ref="MeshA">
            <TresBoxGeometry />
            <TresMeshStandardMaterial color="white" />
        </TresMesh>
        <Suspense>
            <Model :speed="speed" :colorParams="colorParams" :persent="persent" />
        </Suspense>

        <!-- 添加顶部的灯光 -->
        <Plane ref="plane1" :position="[0, 1.5, 0]" :scale="[1.2, 2.2, 0]">
            <TresMeshToonMaterial :side="THREE.BackSide" color="white" />
        </Plane>
        <!-- 后期处理效果1 -->
        <!-- <Suspense> 
            <EffectComposer>
                <UnrealBloom />
            </EffectComposer>
        </Suspense> -->
        <!-- 后期处理效果2 -->
        <!-- <Suspense>
            <EffectComposerPmndrs>
                <SepiaPmndrs :intensity="2" />
            </EffectComposerPmndrs>
        </Suspense> -->

        <!-- 环境贴图HDR -->
        <!-- <Environment>
            <Lightformer :intensity="0.75" :position="[0, 5, -9]" />
            <Lightformer from="ring" :rotation-y="-Math.PI / 2" :scale="[10, 10, 1]" />
        </Environment> -->

        <!-- 灯光 -->
        <TresDirectionalLight :position="[0, 2, 4]" :intensity="1" />
        <TresAmbientLight :intensity="1" color="grey" />
        <TresRectAreaLight :width="1.2" :height="2.2" :intensity="1" color="white" :position="[0, 3, 0]" />
        <!-- <Tres -->
        <!-- 汽车灯光 -->
        <!-- 监听性能 -->
        <StatsGl />
        <!-- 加入雾 -->
        <TresFog color="black" :near="1" :far="12" />

    </TresCanvas>

</template>
<script lang="ts" name="App" setup>
    import { close, start } from './nporgress.ts'
    import * as THREE from 'three';
    import { TresCanvas } from '@tresjs/core';
    import { OrbitControls, Sky, Plane, Stars, ScreenSizer, Environment } from '@tresjs/cientos';
    import { useTresContext } from '@tresjs/core';
    import { useWindowSize } from '@vueuse/core';
    import defineNuxtConfig from './nuxt.config.ts';
    import { useTresContextProvider } from "@tresjs/core";
    import { useRenderLoop } from '@tresjs/core';
    import { onMounted, reactive, ref, setBlockTracking, shallowRef, watch, watchEffect } from 'vue';
    //引入外部的模型组件
    import Model from './components/Model.vue';
    import { AxesHelper, Mesh } from 'three';
    import gsap from 'gsap';
    import { color } from 'three/tsl';
    import { useTexture } from '@tresjs/core'
    // 后期处理效果的导入
    import { EffectComposer, Glitch, UnrealBloom, Halftone, SepiaPmndrs, EffectComposerPmndrs } from '@tresjs/post-processing'
    import { StatsGl } from '@tresjs/cientos'



    // 加载纹理
    const texture = useTexture({
        map: 'public/textures/map.001.png',
    });
    const loadedTexture = texture;
    const persent = ref(0)

    const yRotation = shallowRef(0)
    const MeshA = shallowRef()
    useRenderLoop().onLoop(({ delta }) => {
        yRotation.value += 0.02 * delta
        MeshA.value.rotation.x += 2 * delta
    })

    //为子组件的轮子转动定义速度
    const speed = ref(22)
    const colorParams = ref('0xFF0000')
    const control = ref(0)
    const positions = reactive({ positionx: 12, positiony: 1, positionz: 5 })
    const begin = gsap.to(speed, {
        value: 22,
        duration: 2,
        ease: 'power1.inOut',
        paused: true
    })
    const loadSpeed = gsap.to(speed, {
        value: 0,
        duration: 2.3,
        paused: true
    })
    const camera = ref()
    const plane = ref()
    const loading = ref()
    onMounted(() => {
        // //定义场景
        const MeshToon = ref()
        const plane1 = ref()
        const canvas = ref()
        const timer = setTimeout(() => {
            gsap.to(positions, {
                positionx: 4,
                positiony: 1,
                positionz: 0.6,
                duration: 2,
                ease: 'power1.inOut',
            })
            loadSpeed.play();
            // const sceneList = canvas.value.context.scene.value.children
            (async () => {
                const loadedTexture = await texture;
                // 设置纹理的包裹模式
                loadedTexture.map.wrapS = THREE.RepeatWrapping;
                loadedTexture.map.wrapT = THREE.RepeatWrapping;

                // 计算纹理的重复次数，确保按原比例显示
                const geometry = new THREE.PlaneGeometry(4, 4);
                const aspectRatio = loadedTexture.map.image.width / loadedTexture.map.image.height;
                const planeAspectRatio = geometry.parameters.width / geometry.parameters.height;

                if (aspectRatio > planeAspectRatio) {
                    loadedTexture.map.repeat.set(repeatNumber.value, (aspectRatio / planeAspectRatio) * repeatNumber.value);
                } else {
                    loadedTexture.map.repeat.set((planeAspectRatio / aspectRatio) * repeatNumber.value, repeatNumber.value);
                }
                // 创建 MeshToonMaterial 材质
                const material = new THREE.MeshToonMaterial({ map: loadedTexture.map });
                // console.log(loadedTexture.map)

                plane.value.instance.material = material;
            })();
        }, 600)
    }
    );
    // 使用 async/await 等待 Promise 完成
    // 定义地面纹理的repeat重复次数
    const repeatNumber = ref(6);


    // 鼠标点击事件 车轮旋转
    window.addEventListener('mousedown', () => {
        begin.play()
    })
    window.addEventListener('mouseup', () => {
        begin.reverse()
    })
    window.addEventListener('touchstart', () => {
        begin.play()
    })
    window.addEventListener('touchend', () => {
        begin.reverse()
    })

</script>
<style>
html,
body {
    margin: 0;
    padding: 0;
    height: 100%;
    width: 100%;
}

#app {
    height: 100%;
    width: 100%;
    background-color: #000;
}

.loading {
    width: 100vw;
    height: 100vh;
    background-color: grey;
    position: fixed;
    top: 0;
    left: 0;
    z-index: 9999;
    text-align: center;
    line-height: 100vh;
    font-size: 50px
}
</style>
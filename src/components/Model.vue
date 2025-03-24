<script lang="ts" name="Model" setup>
    import * as THREE from 'three'
    import { TresCanvas } from '@tresjs/core';
    import { MeshGlassMaterial, ScreenSizer, Environment } from '@tresjs/cientos';
    import { useTresContext } from '@tresjs/core';
    import { Mesh, BoxGeometry, MeshBasicMaterial } from 'three'
    import { useRenderLoop } from '@tresjs/core';
    import { useGLTF } from '@tresjs/cientos'
    import { onBeforeMount, defineEmits, defineProps, watchEffect, ref, onMounted, reactive } from 'vue';
    import { useControls, TresLeches } from '@tresjs/leches'
    import '@tresjs/leches/styles'
    import { any, color, string } from 'three/tsl';
    import gsap from 'gsap';
    import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
    import { scene } from '@tresjs/core/dist/src/utils/is.js';


    // 定义组件的props
    const props = defineProps({
        speed: Number,
        colorParams: String,
        persent: Number,
    });
    const ccc = ref(false)
    const wheelList: any = [];
    let objtt = {}
    //加载资源
    const loader = new GLTFLoader();
    loader.load('public/无标题.glb', function (gltf) {
        const wheelList = gltf.scene.children[0].children;
        objtt = gltf.scene.children[0]
        ccc.value = true
        // const nodes = gltf.scene
        // 模型的旋转
        const { onLoop } = useRenderLoop()
        onLoop(({ delta }) => {
            // 轮子的旋转 围绕自身的轴旋转
            wheelList[0].rotation.x += (props.speed || 0) * delta
            wheelList[1].rotation.x += (props.speed || 0) * delta
            wheelList[2].rotation.x += (props.speed || 0) * delta
            wheelList[3].rotation.x += (props.speed || 0) * delta
            // nodes.Car.rotation.y += (props.speed || 0) / 9 * delta
        })
    }, /* 后面是加载进度函数*/(e) => {
        const loadera = (document.querySelector('.loading') as any)
        loadera.innerHTML = `加载中：${Math.floor(e.loaded / e.total * 100)}%`
        if (e.loaded / e.total * 100 == 100) {
            setTimeout(() => {
                loadera.style.display = 'none'
            }, 800)
        }
    }) //资源加载完毕


</script>
<template>
    <primitive v-if="ccc" :object="objtt" :position="[0, 0, 0]" />
    <TresRectAreaLight :position="[0, 10, 0]" :width="10" :height="10" :intensity="220" />
    <Suspense>
        <Environment files="/public/textures/golden_gate_hills_2k.hdr" />
    </Suspense>
</template>
<style scoped></style>
<template>
  <v-container>
    <canvas id="webgl"></canvas>

    <span id="scrollProgress"></span>

    <main>
      <h1>3Dスクロールアニメーション</h1>
      <section>
        <h2>スクロールを開始してください</h2>
      </section>
      <section>
        <h2>オブジェクトの位置が変わります</h2>
        <p>ボックスの位置が変わっています</p>
      </section>
      <section>
        <h2>オブジェクトの位置が変わります</h2>
        <p>ボックスの位置が変わっています</p>
      </section>
      <section>
        <h2>オブジェクトの位置が変わります</h2>
        <p>カメラの位置が変わっています</p>
      </section>

      <section>
        <h2>一番下にいます</h2>
        <p>立方体が自動回転しています</p>
        <p>上にスクロールしてアニメーションを反転できます</p>
      </section>
    </main>
  </v-container>
</template>

<script lang="ts">
import * as THREE from "three";

export default {
  mounted() {

    /**
     * 必須の3要素
     */
    // Canvas
    const canvas = document.querySelector("#webgl") as HTMLCanvasElement;


    //Sizes
    const sizes = {
      width: window.innerWidth,
      height: window.innerHeight,
    };

    // Scene
    const scene = new THREE.Scene();

    /**
     * GridHelperの設定
     */
    const gridHelper = new THREE.GridHelper(30, 30);
    scene.add(gridHelper);

    // Camera
    const camera = new THREE.PerspectiveCamera(
      75,
      sizes.width / sizes.height,
      0.1,
      100
    );
    camera.position.z = 6;
    scene.add(camera);

    //Renderer
    const renderer = new THREE.WebGLRenderer({
      canvas: canvas,
    });
    renderer.setSize(sizes.width, sizes.height);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

    //オブジェクトの追加
    const geometry = new THREE.BoxGeometry(5, 5, 5, 10);
    const material = new THREE.MeshNormalMaterial();

    const torus = new THREE.Mesh(geometry, material);
    torus.position.set(0, 0.5, -15);
    torus.rotation.set(1, 1, 0);
    scene.add(torus);

    /**
     * 線形補間
     * lerp(min, max, ratio)
     * eg,
     * lerp(20, 60, .5)) = 40
     * lerp(-20, 60, .5)) = 20
     * lerp(20, 60, .75)) = 50
     * lerp(-20, -10, .1)) = -.19
     */
    function lerp(x: number, y: number, a: number) {
      return (1 - a) * x + a * y;
    }


    /**
     * 特定のスクロール率で開始、終了
     **/
    function scaleParcent(start: number, end: number) {
      return (scrollPercent - start) / (end - start);
    }

    /**
     * 関数用の空の配列を準備
     */
    const animationScripts: { start: number; end: number; function: () => void }[] = [];


    /**
     * スクロールアニメーション
     */
    animationScripts.push({
      start: 0,
      end: 40,
      function() {
        camera.lookAt(torus.position);
        camera.position.set(0, 1, 10);
        torus.position.z = lerp(-10, 2, scaleParcent(0, 40));
        // console.log(torus.position.z);
      },
    });
    // console.log(animationScripts);

    animationScripts.push({
      start: 40,
      end: 60,
      function() {
        camera.lookAt(torus.position);
        camera.position.set(0, 1, 10);
        torus.rotation.z = lerp(0, Math.PI, scaleParcent(40, 60));
        // console.log(torus.position.z);
      },
    });

    animationScripts.push({
      start: 60,
      end: 80,
      function() {
        camera.lookAt(torus.position);
        camera.position.x = lerp(0, 10, scaleParcent(60, 80));
        camera.position.y = lerp(1, 12, scaleParcent(60, 80));
        camera.position.z = lerp(10, 20, scaleParcent(60, 80));
        // console.log(torus.position.z);
      },
    });

    animationScripts.push({
      start: 80,
      end: 101,
      function() {
        camera.lookAt(torus.position);
        torus.rotation.x += 0.02;
        torus.rotation.y += 0.02;
        // console.log(torus.position.z);
      },
    });

    /**
     * スクロールアニメーション開始
     */
    function playScrollAnimation() {
      animationScripts.forEach((animation) => {
        if (scrollPercent >= animation.start && scrollPercent < animation.end) {
          animation.function();
        }
      });
    }

    /**
     * ブラウザのスクロール率を導出
     */
    let scrollPercent = 0;

    document.body.onscroll = () => {
      //現在のスクロールの進捗をパーセントで計算する
      scrollPercent =
        (document.documentElement.scrollTop /
          (document.documentElement.scrollHeight -
            document.documentElement.clientHeight)) *
        100;
    };

    //アニメーション
    const tick = () => {
      window.requestAnimationFrame(tick);
      playScrollAnimation();
      renderer.render(scene, camera);
    };

    tick();

    //ブラウザのリサイズ操作
    window.addEventListener("resize", () => {
      sizes.width = window.innerWidth;
      sizes.height = window.innerHeight;

      camera.aspect = sizes.width / sizes.height;
      camera.updateProjectionMatrix();

      renderer.setSize(sizes.width, sizes.height);
      renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    });

    window.scrollTo({ top: 0, behavior: "smooth" });


  },
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: "Poppins", sans-serif;
}

body {
  color: white;
  overflow-x: hidden;
}

canvas {
  position: fixed;
  top: 0;
  left: 0;
}

main {
  position: absolute;
  width: 100vw;
  height: 200vh;
  z-index: 100;
  justify-content: center;
  text-align: center;
  font-size: 2rem;
  margin-top: 100px;
}

section {
  min-height: 100vh;
  padding: 20px;
  font-size: 1.5rem;
}
</style>

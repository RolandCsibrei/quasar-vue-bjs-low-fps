<template>
  <q-page>
    <div class="full-width q-ma-md">
      <q-btn label="Emit Engine" @click="emitEngine" color="red" />
    </div>
    <canvas ref="bjsCanvas" width="1000" height="500" />
  </q-page>
</template>

<script lang="ts">
import { defineComponent, onMounted, ref } from '@vue/composition-api';
import {
  Color4,
  Engine,
  FreeCamera,
  HemisphericLight,
  MeshBuilder,
  Scene,
  Vector3
} from '@babylonjs/core';
import {
  AdvancedDynamicTexture,
  Control,
  Rectangle,
  TextBlock
} from '@babylonjs/gui';

export default defineComponent({
  name: 'PageIndex',
  setup(_, { emit }) {
    let engine: Engine | null = null;
    const bjsCanvas = ref<HTMLCanvasElement | null>(null);
    onMounted(() => {
      const canvas = bjsCanvas.value;
      engine = new Engine(canvas);
      const scene = new Scene(engine);

      scene.clearColor = new Color4(0, 0.1, 0.3, 1);

      // eslint-disable-next-line @typescript-eslint/no-unused-vars
      const light = new HemisphericLight('light', Vector3.Down(), scene);

      const camera = new FreeCamera('Camera', new Vector3(0, 10, 10), scene);
      camera.setTarget(Vector3.ZeroReadOnly);
      camera.speed = 0.4;
      camera.attachControl(canvas, true);

      let count = 1000;
      while (count-- > 0) {
        const box = MeshBuilder.CreateBox(`box${count}`, {}, scene);
        box.position.x = Math.random() * 10 - 20;
        box.position.y = Math.random() * 10 - 20;
        box.position.z = Math.random() * 10 - 20;
      }

      //
      const guiTexture = AdvancedDynamicTexture.CreateFullscreenUI(
        'UI',
        true,
        scene
      );

      const rect1 = new Rectangle();
      rect1.width = 0.1;
      rect1.height = '80px';
      rect1.cornerRadius = 4;
      rect1.color = 'Orange';
      rect1.thickness = 0;
      rect1.background = 'black';
      rect1.horizontalAlignment = Control.HORIZONTAL_ALIGNMENT_LEFT;
      rect1.verticalAlignment = Control.VERTICAL_ALIGNMENT_TOP;
      rect1.name = 'fps';
      guiTexture.addControl(rect1);

      const label = new TextBlock();
      label.text = '0';
      label.color = '#ffffff';
      label.name = 'fpslabel';

      rect1.addControl(label);

      scene.onBeforeRenderObservable.add(() => {
        label.text = engine.getFps().toFixed();
      });

      //

      engine.runRenderLoop(() => {
        scene.render();
      });
    });

    const emitEngine = () => {
      emit('engine', engine);
    };

    return { bjsCanvas, emitEngine };
  }
});
</script>

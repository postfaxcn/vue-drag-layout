<!--
 * @Author: Yanshan Lin yanshan.lin@happyelements.com
 * @Date: 2022-12-30 14:24:16
 * @LastEditors: Yanshan Lin yanshan.lin@happyelements.com
 * @LastEditTime: 2023-01-05 09:04:00
 * @FilePath: /drag-resize/src/App.vue
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
<template>
  <div>
    <div class="list" ref="listEl">
      <VueDragLayout
        v-for="(rect, index) in rects"
        :key="index"
        :id="rect.id"
        :w="rect.width"
        :h="rect.height"
        :x="rect.left"
        :y="rect.top"
        :parentW="listWidth"
        :parentH="listHeight"
        :axis="rect.axis"
        :isActive="rect.active"
        :minw="rect.minw"
        :minh="rect.minh"
        :isDraggable="rect.draggable"
        :isResizable="rect.resizable"
        :parentLimitation="rect.parentLim"
        :snapToGrid="rect.snapToGrid"
        :aspectRatio="rect.aspectRatio"
        :z="rect.zIndex"
        :contentClass="rect.class"
        @activated="activateEv(index)"
        @deactivated="deactivateEv(index)"
        @dragging="changePosition($event, index)"
        @resizing="changeSize($event, index)"
      >
        <div class="filler" :style="{ backgroundColor: rect.color }"></div>
      </VueDragLayout>
    </div>
    <toolbar :value="active"></toolbar>
  </div>
</template>

<script>
import VueDragLayout from "./components/Layout.vue";
import toolbar from "./components/toolbar.vue";
import { onMounted, ref } from "vue";
export default {
  name: "App",
  components: {
    VueDragLayout,
    toolbar,
  },
  setup() {
    const listWidth = ref(0);
    const listHeight = ref(0);
    const listEl = ref();
    const active = ref({});

    const rects = ref([
      {
        id: "1",
        width: 200,
        height: 150,
        top: 10,
        left: 10,
        draggable: true,
        resizable: true,
        minw: 10,
        minh: 10,
        axis: "both",
        parentLim: true,
        snapToGrid: false,
        aspectRatio: false,
        zIndex: 1,
        color: "#EF9A9A",
        active: false,
      },
      {
        id: "2",
        width: 200,
        height: 150,
        top: 10,
        left: 220,
        draggable: true,
        resizable: true,
        minw: 10,
        minh: 10,
        axis: "both",
        parentLim: true,
        snapToGrid: false,
        aspectRatio: false,
        zIndex: 2,
        color: "#AED581",
        active: false,
      },
      {
        id: "3",
        width: 200,
        height: 150,
        top: 170,
        left: 10,
        draggable: true,
        resizable: true,
        minw: 10,
        minh: 10,
        axis: "both",
        parentLim: true,
        snapToGrid: false,
        aspectRatio: false,
        zIndex: 3,
        color: "#81D4FA",
        active: false,
      },
    ]);

    const activateEv = (index) => {
      rects.value.forEach((rect, idx) => (rect.active = index === idx));
      active.value = rects.value[index];
    };

    const deactivateEv = (index) => {
      rects.value[index].active = false;
    };

    const changePosition = (newRect, index) => {
      rects.value[index].top = newRect.top;
      rects.value[index].left = newRect.left;
      rects.value[index].width = newRect.width;
      rects.value[index].height = newRect.height;
    };

    const changeSize = (newRect, index) => {
      rects.value[index].top = newRect.top;
      rects.value[index].left = newRect.left;
      rects.value[index].width = newRect.width;
      rects.value[index].height = newRect.height;
    };

    const onToolAction = () => {};

    onMounted(() => {
      listWidth.value = listEl.value.clientWidth;
      listHeight.value = listEl.value.clientHeight;

      addEventListener("resize", () => {
        listWidth.value = listEl.value.clientWidth;
        listHeight.value = listEl.value.clientHeight;
      });
    });

    return {
      listEl,
      listWidth,
      listHeight,
      rects,
      activateEv,
      deactivateEv,
      changePosition,
      changeSize,
      active,
      onToolAction,
    };
  },
};
</script>

<style lang="scss">
body {
  height: 100vh;
  width: 100vw;
  margin: 0;
  padding: 0;
  background-color: #ececec;
}

#app {
  box-sizing: border-box;
  width: 100%;
  height: 100%;
  position: relative;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.list {
  position: absolute;
  top: 30px;
  bottom: 30px;
  left: 30px;
  right: 300px;
  box-shadow: 0 0 2px #aaa;
  background-color: white;
}

.box-shaddow {
  box-shadow: 10px 10px 15px 0px rgba(125, 125, 125, 1);
}
.filler {
  width: 100%;
  height: 100%;
  display: inline-block;
  position: absolute;
}
</style>

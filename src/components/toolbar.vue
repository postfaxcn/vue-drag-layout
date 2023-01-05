<!--
 * @Author: Yanshan Lin yanshan.lin@happyelements.com
 * @Date: 2023-01-04 14:15:36
 * @LastEditors: Yanshan Lin yanshan.lin@happyelements.com
 * @LastEditTime: 2023-01-04 17:31:19
 * @FilePath: /drag-resize/src/components/Toolbar.vue
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
<template>
  <div class="toolbar" @mousedown.stop>
    <div class="toolbar-wh-row">
      <p class="toolbar-row-title">Position</p>
      <span class="toolbar-position-inp">
        top
        <input :value="top" @keyup="changeTop" />
        <svg
          version="1.1"
          viewBox="0 0 100 100"
          class="position-lock-icon svg-icon svg-fill"
          style="width: 15px; height: 15px"
          @click.native="toggleYLock"
        >
          <path
            :fill="topIsLocked ? '#42b983' : '#AAA'"
            stroke="none"
            pid="0"
            d="M33.333 50H40v13.334h-6.667zM46.667 50h6.666v13.334h-6.666zM60 50h6.667v13.334H60z"
          ></path>
          <path
            :fill="topIsLocked ? '#35495e' : '#AAA'"
            stroke="none"
            pid="1"
            d="M73.333 36.667H70V30c0-11.045-8.955-20-20-20s-20 8.955-20 20v6.667h-3.333c-3.666 0-6.667 3-6.667 6.667v40C20 86.999 23.001 90 26.667 90h46.666C76.999 90 80 86.999 80 83.333v-40c0-3.666-3.001-6.666-6.667-6.666zM36.667 30c0-7.363 5.97-13.333 13.333-13.333S63.333 22.637 63.333 30v6.667H36.667V30zm36.666 13.333v26.664H26.667V43.333h46.666zm-46.666 40v-6.666h46.666v6.666H26.667z"
          ></path>
        </svg>
      </span>
      <span class="toolbar-position-inp">
        left
        <input :value="left" @keyup="changeLeft" />
        <svg
          version="1.1"
          viewBox="0 0 100 100"
          class="position-lock-icon svg-icon svg-fill"
          style="width: 15px; height: 15px"
          @click.native="toggleXLock"
        >
          <path
            :fill="leftIsLocked ? '#42b983' : '#AAA'"
            stroke="none"
            pid="0"
            d="M33.333 50H40v13.334h-6.667zM46.667 50h6.666v13.334h-6.666zM60 50h6.667v13.334H60z"
          ></path>
          <path
            :fill="leftIsLocked ? '#35495e' : '#AAA'"
            stroke="none"
            pid="1"
            d="M73.333 36.667H70V30c0-11.045-8.955-20-20-20s-20 8.955-20 20v6.667h-3.333c-3.666 0-6.667 3-6.667 6.667v40C20 86.999 23.001 90 26.667 90h46.666C76.999 90 80 86.999 80 83.333v-40c0-3.666-3.001-6.666-6.667-6.666zM36.667 30c0-7.363 5.97-13.333 13.333-13.333S63.333 22.637 63.333 30v6.667H36.667V30zm36.666 13.333v26.664H26.667V43.333h46.666zm-46.666 40v-6.666h46.666v6.666H26.667z"
          ></path>
        </svg>
      </span>
    </div>
    <div class="toolbar-wh-row">
      <p class="toolbar-row-title">Size</p>
      <span class="toolbar-size-inp">
        width
        <input :value="width" @keyup="changeWidth" />
        <svg
          version="1.1"
          viewBox="0 0 100 100"
          class="position-lock-icon svg-icon svg-fill"
          style="width: 15px; height: 15px"
          @click.native="toggleAspect"
        >
          <path
            :fill="aspectRatio ? '#42b983' : '#AAA'"
            stroke="none"
            pid="0"
            d="M33.333 50H40v13.334h-6.667zM46.667 50h6.666v13.334h-6.666zM60 50h6.667v13.334H60z"
          ></path>
          <path
            :fill="aspectRatio ? '#35495e' : '#AAA'"
            stroke="none"
            pid="1"
            d="M73.333 36.667H70V30c0-11.045-8.955-20-20-20s-20 8.955-20 20v6.667h-3.333c-3.666 0-6.667 3-6.667 6.667v40C20 86.999 23.001 90 26.667 90h46.666C76.999 90 80 86.999 80 83.333v-40c0-3.666-3.001-6.666-6.667-6.666zM36.667 30c0-7.363 5.97-13.333 13.333-13.333S63.333 22.637 63.333 30v6.667H36.667V30zm36.666 13.333v26.664H26.667V43.333h46.666zm-46.666 40v-6.666h46.666v6.666H26.667z"
          ></path>
        </svg>
      </span>
      <span class="toolbar-size-inp">
        height
        <input :value="height" @keyup="changeHeight" />
      </span>
    </div>
    <div class="toolbar-wh-row">
      <p class="toolbar-row-title">Minimal size</p>
      <span class="toolbar-size-inp">
        width
        <input
          :value="minw"
          :disabled="activeRect === null"
          @keyup="changeMinWidth"
        />
      </span>
      <span class="toolbar-size-inp">
        height
        <input
          :value="minh"
          :disabled="activeRect === null"
          @keyup="changeMinHeight"
        />
      </span>
    </div>
    <div class="toolbar-wh-row">
      <p class="toolbar-row-title">Restrictions</p>
      <label class="toolbar-check-inp">
        <input
          type="checkbox"
          @change="toggleResizable"
          :checked="resizable"
          :disabled="activeRect === null"
        />
        isResizable
      </label>
      <label class="toolbar-check-inp">
        <input
          type="checkbox"
          @change="toggleDraggable"
          :checked="draggable"
          :disabled="activeRect === null"
        />
        isDraggable
      </label>
      <label class="toolbar-check-inp">
        <input
          type="checkbox"
          @change="toggleSnapToGrid"
          :checked="snapToGrid"
          :disabled="activeRect === null"
        />
        snapToGrid
      </label>
      <label class="toolbar-check-inp">
        <input
          type="checkbox"
          @change="toggleParentLimitation"
          :checked="parentLim"
          :disabled="activeRect === null"
        />
        parentLimitation
      </label>
      <svg
        version="1.1"
        viewBox="0 0 100 100"
        class="to-bottom-icon svg-icon svg-fill"
        style="width: 30px; height: 30px"
        @click.native="toBottom"
      >
        <path
          :fill="activeRect && zIndex !== 'isFirst' ? '#35495e' : '#AAA'"
          stroke="none"
          pid="0"
          d="M73.333 66.666v-10c0-1.832-1.5-3.332-3.333-3.332H30a3.345 3.345 0 0 0-3.333 3.332v10H20v6.668h60v-6.668h-6.667zM66.667 40V30a3.344 3.344 0 0 0-3.334-3.333H36.667A3.344 3.344 0 0 0 33.333 30v10H20v6.667h60V40H66.667zM60 40H40v-6.667h20V40z"
        ></path>
      </svg>
      <svg
        version="1.1"
        viewBox="0 0 100 100"
        class="to-top-icon svg-icon svg-fill"
        style="width: 30px; height: 30px"
        @click.native="toTop"
      >
        <path
          :fill="activeRect && zIndex !== 'isFirst' ? '#35495e' : '#AAA'"
          stroke="none"
          pid="0"
          d="M80 53.334H20V60h6.667v10c0 1.836 1.5 3.334 3.333 3.334h40A3.344 3.344 0 0 0 73.333 70V60H80v-6.666zM20 26.667v6.667h13.333v10a3.344 3.344 0 0 0 3.334 3.333h26.666a3.344 3.344 0 0 0 3.334-3.333v-10H80v-6.667H20zM60 40H40v-6.667h20V40z"
        ></path>
      </svg>
    </div>
  </div>
</template>

<script>
import { computed } from "vue";
export default {
  props: {
    value: {
      type: Object,
    },
  },
  setup(props) {
    const activeRect = computed(() => props.value);
    const top = computed(() => props.value.top);
    const left = computed(() => props.value.left);
    const width = computed(() => props.value.width);
    const height = computed(() => props.value.height);
    const minw = computed(() => props.value.minw);
    const minh = computed(() => props.value.minh);
    const resizable = computed(() => props.value.resizable);
    const draggable = computed(() => props.value.draggable);
    const snapToGrid = computed(() => props.value.snapToGrid);
    const parentLim = computed(() => props.value.parentLim);
    const zIndex = computed(() =>
      props.value.zIndex === 1 ? "isFirst" : "isLast"
    );
    const topIsLocked = computed(() => {
      if (!activeRect.value) {
        return false;
      }
      return activeRect.value.axis === "x" || activeRect.value.axis === "none";
    });
    const leftIsLocked = computed(() => {
      if (!activeRect.value) {
        return false;
      }
      return activeRect.value.axis === "y" || activeRect.value.axis === "none";
    });
    const aspectRatio = computed(() => {
      return activeRect.value.aspectRatio;
    });
    const changeLeft = (event) => {
      let left = parseInt(event.target.value);
      if (typeof left !== "number" || isNaN(left)) {
        left = activeRect.value.left;
        event.target.value = left;
      }
      activeRect.value.left = left;
    };
    const changeTop = (event) => {
      let top = parseInt(event.target.value);
      if (typeof top !== "number" || isNaN(top)) {
        top = activeRect.value.top;
        event.target.value = top;
        return;
      }
      activeRect.value.top = top;
    };
    const changeWidth = (event) => {
      let width = parseInt(event.target.value);
      if (typeof width !== "number" || isNaN(width)) {
        width = activeRect.value.width;
        event.target.value = width;
      }
      activeRect.value.width = width;
    };
    const changeHeight = (event) => {
      let height = parseInt(event.target.value);
      if (typeof height !== "number" || isNaN(height)) {
        height = activeRect.value.height;
        event.target.value = height;
      }
      activeRect.value.height = height;
    };
    const changeMinHeight = (event) => {
      let minh = parseInt(event.target.value);
      if (typeof minh !== "number" || isNaN(minh)) {
        minh = 1;
      }
      if (minh <= 0) {
        minh = 1;
      } else if (minh > activeRect.value.height) {
        minh = activeRect.value.height;
      }
      event.target.value = minh;
      activeRect.value.minh = minh;
    };
    const changeMinWidth = (event) => {
      let minw = parseInt(event.target.value);
      if (typeof minw !== "number" || isNaN(minw)) {
        minw = 1;
      }
      if (minw <= 0) {
        minw = 1;
      } else if (minw > activeRect.value.width) {
        minw = activeRect.value.width;
      }
      event.target.value = minw;
      activeRect.value.minw = minw;
    };
    const changeXLock = (axis) => {
      let out = "";
      switch (axis) {
        case "both":
          out = "y";
          break;
        case "x":
          out = "none";
          break;
        case "y":
          out = "both";
          break;
        case "none":
          out = "x";
          break;
      }
      return out;
    };
    const changeYLock = (axis) => {
      let out = "";
      switch (axis) {
        case "both":
          out = "x";
          break;
        case "x":
          out = "both";
          break;
        case "y":
          out = "none";
          break;
        case "none":
          out = "y";
          break;
      }
      return out;
    };
    const toggleXLock = () => {
      if (!activeRect.value) {
        return;
      }
      activeRect.value.axis = changeXLock(activeRect.value.axis);
    };
    const toggleYLock = () => {
      if (!activeRect.value) {
        return;
      }
      activeRect.value.axis = changeYLock(activeRect.value.axis);
    };
    const toggleSnapToGrid = () => {
      activeRect.value.snapToGrid = !activeRect.value.snapToGrid;
    };
    const toggleAspect = () => {
      if (!activeRect.value) {
        return;
      }
      activeRect.value.aspectRatio = !activeRect.value.aspectRatio;
    };
    const toBottom = () => {
      activeRect.value.zIndex -= 1;
    };
    const toTop = () => {
      activeRect.value.zIndex += 1;
    };
    return {
      activeRect,
      top,
      left,
      width,
      height,
      minw,
      minh,
      resizable,
      draggable,
      snapToGrid,
      parentLim,
      zIndex,
      topIsLocked,
      leftIsLocked,
      aspectRatio,
      changeLeft,
      changeTop,
      changeWidth,
      changeHeight,
      changeMinHeight,
      changeMinWidth,
      toggleXLock,
      toggleYLock,
      toggleSnapToGrid,
      toggleAspect,
      toBottom,
      toTop,
    };
  },
};
</script>

<style lang="scss" scoped>
.toolbar {
  position: absolute;
  right: 0;
  top: 0;
  width: 220px;
  padding: 15px 15px 0 15px;
  box-shadow: 0 0 2px #aaa;
  box-sizing: border-box;
  background-color: white;
  margin: 30px 30px 30px 0;
}
.toolbar-wh-row {
  margin-bottom: 20px;
}
.toolbar-row-title {
  width: 220px;
  font-size: 14px;
  font-family: "Lato", sans-serif;
  font-weight: 500;
  margin: 0 0 3px 0;
  color: #1a173b;
}
.toolbar-position-inp,
.toolbar-size-inp {
  width: 90px;
  font-size: 11px;
  color: #bbb;
  font-weight: 300;
  display: inline-block;
  position: relative;
}

.toolbar-size-inp input,
.toolbar-position-inp input {
  width: 70px;
  display: inline-block;
  border: 1px solid #bfbfca;
  margin-top: 2px;
  height: 16px;
}

.toolbar-size-inp input[disabled],
.toolbar-position-inp input[disabled] {
  border: 1px solid #dcdce7;
  color: #aaaaaa;
}

.position-lock-icon,
.size-lock-icon {
  position: absolute;
  bottom: 3px;
  right: 17px;
  cursor: pointer;
}
.size-lock-icon {
  bottom: 2px;
  right: -3px;
}

.toolbar-check-inp {
  color: #445477;
  font-size: 13px;
  width: 180px;
  display: inline-block;
  margin: 2px 0;
}
.toolbar-row-title + label {
  margin-top: 5px;
}
.toolbar-check-inp input {
  border: 1px solid #bfbfca;
}
.to-top-icon,
.to-bottom-icon {
  margin: 10px 30px;
  cursor: pointer;
}
</style>

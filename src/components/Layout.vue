<!--
 * @Author: Yanshan Lin yanshan.lin@happyelements.com
 * @Date: 2022-12-30 14:26:07
 * @LastEditors: Yanshan Lin yanshan.lin@happyelements.com
 * @LastEditTime: 2023-01-05 09:08:05
 * @FilePath: /drag-resize/src/components/Grid.vue
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
<template>
  <div
    ref="parentElement"
    class="vdr"
    :style="positionStyle"
    :class="`${active || isActive ? 'active' : 'inactive'} ${
      contentClass ? contentClass : ''
    }`"
    @mousedown="bodyDown($event)"
    @touchstart="bodyDown($event)"
    @touchend="up($event)"
  >
    <div :style="sizeStyle" class="content-container" ref="container">
      <slot></slot>
    </div>
    <div
      v-for="(stick, index) in sticks"
      :key="index"
      class="vdr-stick"
      :class="['vdr-stick-' + stick, isResizable ? '' : 'not-resizable']"
      @mousedown.stop.prevent="stickDown(stick, $event)"
      @touchstart.stop.prevent="stickDown(stick, $event)"
      :style="vdrStick(stick)"
    ></div>
  </div>
</template>

<script>
import {
  ref,
  onMounted,
  onUnmounted,
  reactive,
  computed,
  nextTick,
  watch,
  watchEffect,
} from "@vue/runtime-core";
export default {
  name: "vue-drag-layout",
  props: {
    id: {
      type: String,
    },
    stickSize: {
      type: Number,
      default: 8,
    },
    parentScaleX: {
      type: Number,
      default: 1,
    },
    parentScaleY: {
      type: Number,
      default: 1,
    },
    isActive: {
      type: Boolean,
      default: false,
    },
    preventActiveBehavior: {
      type: Boolean,
      default: false,
    },
    isDraggable: {
      type: Boolean,
      default: true,
    },
    isResizable: {
      type: Boolean,
      default: true,
    },
    aspectRatio: {
      type: Boolean,
      default: false,
    },
    parentLimitation: {
      type: Boolean,
      default: false,
    },
    snapToGrid: {
      type: Boolean,
      default: false,
    },
    gridX: {
      type: Number,
      default: 50,
      validator(val) {
        return val >= 0;
      },
    },
    gridY: {
      type: Number,
      default: 50,
      validator(val) {
        return val >= 0;
      },
    },
    parentW: {
      type: Number,
      default: 0,
      validator(val) {
        return val >= 0;
      },
    },
    parentH: {
      type: Number,
      default: 0,
      validator(val) {
        return val >= 0;
      },
    },
    w: {
      type: [String, Number],
      default: 200,
      validator(val) {
        return typeof val === "string" ? val === "auto" : val >= 0;
      },
    },
    h: {
      type: [String, Number],
      default: 200,
      validator(val) {
        return typeof val === "string" ? val === "auto" : val >= 0;
      },
    },
    minw: {
      type: Number,
      default: 50,
      validator(val) {
        return val >= 0;
      },
    },
    minh: {
      type: Number,
      default: 50,
      validator(val) {
        return val >= 0;
      },
    },
    x: {
      type: Number,
      default: 0,
      validator(val) {
        return typeof val === "number";
      },
    },
    y: {
      type: Number,
      default: 0,
      validator(val) {
        return typeof val === "number";
      },
    },
    z: {
      type: [String, Number],
      default: "auto",
      validator(val) {
        return typeof val === "string" ? val === "auto" : val >= 0;
      },
    },
    dragHandle: {
      type: String,
      default: null,
    },
    dragCancel: {
      type: String,
      default: null,
    },
    sticks: {
      type: Array,
      default() {
        return ["tl", "tm", "tr", "mr", "br", "bm", "bl", "ml"];
      },
    },
    axis: {
      type: String,
      default: "both",
      validator(val) {
        return ["x", "y", "both", "none"].includes(val);
      },
    },
    contentClass: {
      type: String,
      required: false,
      default: "",
    },
  },
  setup(props, { emit }) {
    const styleMapping = {
      y: {
        t: "top",
        m: "marginTop",
        b: "bottom",
      },
      x: {
        l: "left",
        m: "marginLeft",
        r: "right",
      },
    };

    const addEvents = (events) => {
      events.forEach((cb, eventName) => {
        document.documentElement.addEventListener(eventName, cb);
      });
    };

    const removeEvents = (events) => {
      events.forEach((cb, eventName) => {
        document.documentElement.removeEventListener(eventName, cb);
      });
    };

    // data
    // const fixAspectRatio = ref();
    const parentElement = ref();
    const container = ref();
    const active = ref();
    const zIndex = ref();
    const parentWidth = ref();
    const parentHeight = ref();
    const left = ref();
    const top = ref();
    const right = ref();
    const bottom = ref();
    // const minHeight = ref();
    const aspectFactor = ref();

    const stickDrag = ref(false);
    const bodyDrag = ref(false);
    const dimensionsBeforeMove = reactive({
      pointerX: 0,
      pointerY: 0,
      x: 0,
      y: 0,
      w: 0,
      h: 0,
    });

    const limits = ref({
      left: { min: null, max: null },
      right: { min: null, max: null },
      top: { min: null, max: null },
      bottom: { min: null, max: null },
    });

    const currentStick = ref();

    const positionStyle = computed(() => {
      return {
        top: top.value + "px",
        left: left.value + "px",
        zIndex: zIndex.value,
      };
    });

    const sizeStyle = computed(() => {
      return {
        width: props.w == "auto" ? "auto" : width.value + "px",
        height: props.h == "auto" ? "auto" : height.value + "px",
      };
    });

    const width = computed(() => {
      return parentWidth.value - left.value - right.value;
    });

    const height = computed(() => {
      return parentHeight.value - top.value - bottom.value;
    });

    const rect = computed(() => {
      return {
        left: Math.round(left.value),
        top: Math.round(top.value),
        width: Math.round(width.value),
        height: Math.round(height.value),
      };
    });

    const vdrStick = computed(() => {
      return (stick) => {
        const stickStyle = {
          width: `${props.stickSize / props.parentScaleX}px`,
          height: `${props.stickSize / props.parentScaleY}px`,
        };
        stickStyle[styleMapping.y[stick[0]]] = `${
          props.stickSize / props.parentScaleX / -2
        }px`;
        stickStyle[styleMapping.x[stick[1]]] = `${
          props.stickSize / props.parentScaleX / -2
        }px`;
        return stickStyle;
      };
    });

    const move = (event) => {
      if (!stickDrag.value && !bodyDrag.value) {
        return;
      }

      event.stopPropagation();

      const pageX =
        typeof event.pageX !== "undefined"
          ? event.pageX
          : event.touches[0].pageX;

      const pageY =
        typeof event.pageY !== "undefined"
          ? event.pageY
          : event.touches[0].pageY;

      const delta = {
        x: (dimensionsBeforeMove.pointerX - pageX) / props.parentScaleX,
        y: (dimensionsBeforeMove.pointerY - pageY) / props.parentScaleY,
      };

      if (stickDrag.value) {
        stickMove(delta);
      }

      if (bodyDrag.value) {
        if (props.axis === "x") {
          delta.y = 0;
        } else if (props.axis === "y") {
          delta.x = 0;
        } else if (props.axis === "none") {
          return;
        }
        bodyMove(delta);
      }
    };

    const up = (event) => {
      if (stickDrag.value) {
        stickUp(event);
      } else if (bodyDrag.value) {
        bodyUp(event);
      }
    };

    const deselect = () => {
      if (props.preventActiveBehavior) {
        return;
      }
      active.value = false;
    };

    const bodyDown = (event) => {
      // const { target, button } = event;
      const { button } = event;

      if (!props.preventActiveBehavior) {
        active.value = true;
      }

      if (button && button !== 0) {
        return;
      }

      emit("clicked", event);

      if (!active.value) {
        return;
      }

      if (
        props.dragHandle
        // &&
        // target.getAttribute("data-drag-handle") !== this._uid.toString()
      ) {
        return;
      }

      if (
        props.dragCancel
        // &&
        // target.getAttribute("data-drag-cancel") === this._uid.toString()
      ) {
        return;
      }

      if (typeof event.stopPropagation !== "undefined") {
        event.stopPropagation();
      }

      if (typeof event.preventDefault !== "undefined") {
        event.preventDefault();
      }

      if (props.isDraggable) {
        bodyDrag.value = true;
      }

      const pointerX =
        typeof event.pageX !== "undefined"
          ? event.pageX
          : event.touches[0].pageX;

      const pointerY =
        typeof event.pageY !== "undefined"
          ? event.pageY
          : event.touches[0].pageY;

      saveDimensionsBeforeMove({ pointerX, pointerY });

      if (props.parentLimitation) {
        limits.value = calcDragLimitation();
      }
    };

    const bodyMove = (delta) => {
      let newTop = dimensionsBeforeMove.top - delta.y;
      let newBottom = dimensionsBeforeMove.bottom + delta.y;
      let newLeft = dimensionsBeforeMove.left - delta.x;
      let newRight = dimensionsBeforeMove.right + delta.x;

      if (props.snapToGrid) {
        let alignTop = true;
        let alignLeft = true;

        let diffT = newTop - Math.floor(newTop / props.gridY) * props.gridY;
        let diffB =
          parentHeight.value -
          newBottom -
          Math.floor((parentHeight.value - newBottom) / props.gridY) *
            props.gridY;
        let diffL = newLeft - Math.floor(newLeft / props.gridX) * props.gridX;
        let diffR =
          parentWidth.value -
          newRight -
          Math.floor((parentWidth.value - newRight) / props.gridX) *
            props.gridX;

        if (diffT > props.gridY / 2) {
          diffT -= props.gridY;
        }
        if (diffB > props.gridY / 2) {
          diffB -= props.gridY;
        }
        if (diffL > props.gridX / 2) {
          diffL -= props.gridX;
        }
        if (diffR > props.gridX / 2) {
          diffR -= props.gridX;
        }

        if (Math.abs(diffB) < Math.abs(diffT)) {
          alignTop = false;
        }
        if (Math.abs(diffR) < Math.abs(diffL)) {
          alignLeft = false;
        }

        newTop -= alignTop ? diffT : diffB;
        newBottom = parentHeight.value - height.value - newTop;
        newLeft -= alignLeft ? diffL : diffR;
        newRight = parentWidth.value - width.value - newLeft;
      }

      ({
        newLeft: left.value,
        newRight: right.value,
        newTop: top.value,
        newBottom: bottom.value,
      } = rectCorrectionByLimit({ newLeft, newRight, newTop, newBottom }));

      emit("dragging", rect.value);
    };

    const bodyUp = () => {
      bodyDrag.value = false;
      emit("dragging", rect.value);
      emit("dragstop", rect.value);

      dimensionsBeforeMove.value = {
        pointerX: 0,
        pointerY: 0,
        x: 0,
        y: 0,
        w: 0,
        h: 0,
      };

      limits.value = {
        left: { min: null, max: null },
        right: { min: null, max: null },
        top: { min: null, max: null },
        bottom: { min: null, max: null },
      };
    };

    const stickDown = (stick, event, force = false) => {
      if ((!props.isResizable || !active.value) && !force) {
        return;
      }

      stickDrag.value = true;

      const pointerX =
        typeof event.pageX !== "undefined"
          ? event.pageX
          : event.touches[0].pageX;
      // console.log("pointerX", pointerX);
      const pointerY =
        typeof event.pageY !== "undefined"
          ? event.pageY
          : event.touches[0].pageY;
      // console.log("pointerY", pointerY);
      saveDimensionsBeforeMove({ pointerX, pointerY });

      currentStick.value = stick;

      limits.value = calcResizeLimits();
    };

    const saveDimensionsBeforeMove = ({ pointerX, pointerY }) => {
      dimensionsBeforeMove.pointerX = pointerX;
      dimensionsBeforeMove.pointerY = pointerY;

      dimensionsBeforeMove.left = left.value;
      dimensionsBeforeMove.right = right.value;
      dimensionsBeforeMove.top = top.value;
      dimensionsBeforeMove.bottom = bottom.value;

      dimensionsBeforeMove.width = width.value;
      dimensionsBeforeMove.height = height.value;

      aspectFactor.value = width.value / height.value;
      // console.log("aspectFactor", aspectFactor.value);
    };

    const stickMove = (delta) => {
      const snapToGrid = props.snapToGrid;
      const gridY = props.gridY;
      const gridX = props.gridX;

      let newTop = dimensionsBeforeMove.top;
      let newBottom = dimensionsBeforeMove.bottom;
      let newLeft = dimensionsBeforeMove.left;
      let newRight = dimensionsBeforeMove.right;
      switch (currentStick.value[0]) {
        case "b":
          newBottom = dimensionsBeforeMove.bottom + delta.y;
          if (snapToGrid) {
            newBottom =
              parentHeight.value -
              Math.round((parentHeight.value - newBottom) / gridY) * gridY;
          }
          break;
        case "t":
          newTop = dimensionsBeforeMove.top - delta.y;
          if (snapToGrid) {
            newTop = Math.round(newTop / gridY) * gridY;
          }
          break;
        default:
          break;
      }
      switch (currentStick.value[1]) {
        case "r":
          newRight = dimensionsBeforeMove.right + delta.x;
          if (snapToGrid) {
            newRight =
              parentWidth.value -
              Math.round((parentWidth.value - newRight) / gridX) * gridX;
          }
          break;
        case "l":
          newLeft = dimensionsBeforeMove.left - delta.x;
          if (snapToGrid) {
            newLeft = Math.round(newLeft / gridX) * gridX;
          }
          break;
        default:
          break;
      }

      ({ newLeft, newRight, newTop, newBottom } = rectCorrectionByLimit({
        newLeft,
        newRight,
        newTop,
        newBottom,
      }));

      if (props.aspectRatio) {
        ({ newLeft, newRight, newTop, newBottom } = rectCorrectionByAspectRatio(
          {
            newLeft,
            newRight,
            newTop,
            newBottom,
          }
        ));
      }

      left.value = newLeft;
      right.value = newRight;
      top.value = newTop;
      bottom.value = newBottom;

      emit("resizing", rect.value);
    };

    const stickUp = () => {
      stickDrag.value = false;
      dimensionsBeforeMove.value = {
        pointerX: 0,
        pointerY: 0,
        x: 0,
        y: 0,
        w: 0,
        h: 0,
      };
      limits.value = {
        left: { min: null, max: null },
        right: { min: null, max: null },
        top: { min: null, max: null },
        bottom: { min: null, max: null },
      };
      emit("resizing", rect.value);
      emit("resizestop", rect.value);
    };

    const calcDragLimitation = () => {
      return {
        left: { min: 0, max: parentWidth.value - width.value },
        right: { min: 0, max: parentWidth.value - width.value },
        top: { min: 0, max: parentHeight.value - height.value },
        bottom: { min: 0, max: parentHeight.value - height.value },
      };
    };

    const calcResizeLimits = () => {
      // const aspectFactor = aspectFactor.value;

      let minHeight = props.minh;
      let minWidth = props.minw;

      const parentLim = props.parentLimitation ? 0 : null;

      if (props.aspectRatio) {
        if (minWidth / minHeight > aspectFactor.value) {
          minHeight = minWidth / aspectFactor.value;
        } else {
          minWidth = aspectFactor.value * minHeight;
        }
      }

      const limits = {
        left: { min: parentLim, max: left.value + (width.value - minWidth) },
        right: { min: parentLim, max: right.value + (width.value - minWidth) },
        top: { min: parentLim, max: top.value + (height.value - minHeight) },
        bottom: {
          min: parentLim,
          max: bottom.value + (height.value - minHeight),
        },
      };

      if (props.aspectRatio) {
        const aspectLimits = {
          left: {
            min:
              left.value -
              Math.min(top.value, bottom.value) * aspectFactor.value * 2,
            max:
              left.value +
              ((height.value - minHeight) / 2) * aspectFactor.value * 2,
          },
          right: {
            min:
              right.value -
              Math.min(top.value, bottom.value) * aspectFactor.value * 2,
            max:
              right.value +
              ((height.value - minHeight) / 2) * aspectFactor.value * 2,
          },
          top: {
            min:
              top.value -
              (Math.min(left.value, right.value) / aspectFactor.value) * 2,
            max:
              top.value +
              ((width.value - minWidth) / 2 / aspectFactor.value) * 2,
          },
          bottom: {
            min:
              bottom.value -
              (Math.min(left.value, right.value) / aspectFactor.value) * 2,
            max:
              bottom.value +
              ((width.value - minWidth) / 2 / aspectFactor.value) * 2,
          },
        };

        if (currentStick.value[0] === "m") {
          limits.left = {
            min: Math.max(limits.left.min, aspectLimits.left.min),
            max: Math.min(limits.left.max, aspectLimits.left.max),
          };
          limits.right = {
            min: Math.max(limits.right.min, aspectLimits.right.min),
            max: Math.min(limits.right.max, aspectLimits.right.max),
          };
        } else if (currentStick.value[1] === "m") {
          limits.top = {
            min: Math.max(limits.top.min, aspectLimits.top.min),
            max: Math.min(limits.top.max, aspectLimits.top.max),
          };
          limits.bottom = {
            min: Math.max(limits.bottom.min, aspectLimits.bottom.min),
            max: Math.min(limits.bottom.max, aspectLimits.bottom.max),
          };
        }
      }
      return limits;
    };

    const sideCorrectionByLimit = (limit, current) => {
      let value = current;
      if (limit.min !== null && current < limit.min) {
        value = limit.min;
      } else if (limit.max !== null && limit.max < current) {
        value = limit.max;
      }
      return value;
    };

    const rectCorrectionByLimit = (rect) => {
      let { newRight, newLeft, newBottom, newTop } = rect;
      // console.log("rect", rect);
      newLeft = sideCorrectionByLimit(limits.value.left, newLeft);
      newRight = sideCorrectionByLimit(limits.value.right, newRight);
      newTop = sideCorrectionByLimit(limits.value.top, newTop);
      newBottom = sideCorrectionByLimit(limits.value.bottom, newBottom);

      return {
        newLeft,
        newRight,
        newTop,
        newBottom,
      };
    };

    const rectCorrectionByAspectRatio = (rect) => {
      let { newLeft, newRight, newTop, newBottom } = rect;

      const parentWidth = parentWidth.value;
      const parentHeight = parentHeight.value;
      const aspectFactor = aspectFactor.value;

      let newWidth = parentWidth - newLeft - newRight;
      let newHeight = parentHeight - newTop - newBottom;

      if (currentStick.value[1] === "m") {
        const deltaHeight = newHeight - dimensionsBeforeMove.height;

        newLeft -= (deltaHeight * aspectFactor) / 2;
        newRight -= (deltaHeight * aspectFactor) / 2;
      } else if (currentStick.value[0] === "m") {
        const deltaWidth = newWidth - dimensionsBeforeMove.width;

        newTop -= deltaWidth / aspectFactor / 2;
        newBottom -= deltaWidth / aspectFactor / 2;
      } else if (newWidth / newHeight > aspectFactor) {
        newWidth = aspectFactor * newHeight;

        if (currentStick.value[1] === "l") {
          newLeft = parentWidth - newRight - newWidth;
        } else {
          newRight = parentWidth - newLeft - newWidth;
        }
      } else {
        newHeight = newWidth / aspectFactor;

        if (currentStick.value[0] === "t") {
          newTop = parentHeight - newBottom - newHeight;
        } else {
          newBottom = parentHeight - newTop - newHeight;
        }
      }

      return { newLeft, newRight, newTop, newBottom };
    };

    const domEvents = new Map([
      ["mousemove", move],
      ["mouseup", up],
      ["mouseleave", up],
      ["mousedown", deselect],
      ["touchmove", move],
      ["touchend", up],
      ["touchcancel", up],
      ["touchstart", up],
    ]);

    const init = () => {
      watch(
        () => active.value,
        (isActive) => {
          if (isActive) {
            emit("activated");
          } else {
            emit("deactivated");
          }
        }
      );
      watch(
        () => props.isActive,
        (value) => {
          active.value = value;
        },
        {
          immediate: true,
        }
      );
      watch(
        () => props.z,
        (value) => {
          if (value >= 0 || value === "auto") {
            zIndex.value = value;
          }
        },
        {
          immediate: true,
        }
      );
      watch(
        () => props.x,
        (newVal, oldVal) => {
          if (stickDrag.value || bodyDrag.value || newVal === left.value) {
            return;
          }

          const delta = oldVal - newVal;

          bodyDown({ pageX: left.value, pageY: top.value });
          bodyMove({ x: delta, y: 0 });

          nextTick(() => {
            bodyUp();
          });
        }
      );
      watch(
        () => props.y,
        (newVal, oldVal) => {
          if (stickDrag.value || bodyDrag.value || newVal === top.value) {
            return;
          }

          const delta = oldVal - newVal;

          bodyDown({ pageX: left.value, pageY: top.value });
          bodyMove({ x: 0, y: delta });

          nextTick(() => {
            bodyUp();
          });
        }
      );
      watch(
        () => props.w,
        (newVal, oldVal) => {
          if (stickDrag.value || bodyDrag.value || newVal === width.value) {
            return;
          }

          const stick = "mr";
          const delta = oldVal - newVal;

          stickDown(
            stick,
            { pageX: right.value, pageY: top.value + height.value / 2 },
            true
          );
          stickMove({ x: delta, y: 0 });

          nextTick(() => {
            stickUp();
          });
        }
      );
      watch(
        () => props.h,
        (newVal, oldVal) => {
          if (stickDrag.value || bodyDrag.value || newVal === height.value) {
            return;
          }

          const stick = "bm";
          const delta = oldVal - newVal;

          stickDown(
            stick,
            { pageX: left.value + width.value / 2, pageY: bottom.value },
            true
          );
          stickMove({ x: 0, y: delta });

          nextTick(() => {
            stickUp();
          });
        }
      );
      watch(
        () => props.parentW,
        (value) => {
          right.value = value - width.value - left.value;
          parentWidth.value = value;
        }
      );
      watch(
        () => props.parentH,
        (value) => {
          bottom.value = value - height.value - top.value;
          parentHeight.value = value;
        }
      );
    };

    watchEffect(() => {
      parentWidth.value = props.parentW
        ? props.parentW
        : parentElement?.value?.clientWidth;
      parentHeight.value = props.parentH
        ? props.parentH
        : parentElement?.value?.clientHeight;

      left.value = props.x;
      top.value = props.y;
      right.value =
        parentWidth.value -
        (props.w === "auto" ? container?.value?.scrollWidth : props.w) -
        left.value;
      bottom.value =
        parentHeight.value -
        (props.h === "auto" ? container?.value?.scrollHeight : props.h) -
        top.value;
    });

    onMounted(() => {
      addEvents(domEvents);
      init();
      // window.addEventListener("resize", () => {
      //   bodyMove({ x: 0, y: 0 });
      // });
    });

    onUnmounted(() => {
      removeEvents(domEvents);
    });

    return {
      parentElement,
      container,
      positionStyle,
      bodyDown,
      up,
      active,
      sizeStyle,
      stickDown,
      vdrStick,
    };
  },
};
</script>

<style lang="scss" scoped>
.vdr {
  position: absolute;
  box-sizing: border-box;
}
.vdr.active:before {
  content: "";
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  box-sizing: border-box;
  outline: 1px dashed #d6d6d6;
}
.vdr-stick {
  box-sizing: border-box;
  position: absolute;
  font-size: 1px;
  background: #ffffff;
  border: 1px solid #6c6c6c;
  box-shadow: 0 0 2px #bbb;
}
.inactive .vdr-stick {
  display: none;
}
.vdr-stick-tl,
.vdr-stick-br {
  cursor: nwse-resize;
}
.vdr-stick-tm,
.vdr-stick-bm {
  left: 50%;
  cursor: ns-resize;
}
.vdr-stick-tr,
.vdr-stick-bl {
  cursor: nesw-resize;
}
.vdr-stick-ml,
.vdr-stick-mr {
  top: 50%;
  cursor: ew-resize;
}
.vdr-stick.not-resizable {
  display: none;
}
.content-container {
  display: block;
  position: relative;
}
</style>

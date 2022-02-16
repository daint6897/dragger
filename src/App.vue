<template>
  <div id="app">
    <div class="content_wrap xxxxx">
      <div class="content_left">
        <div
            @drag="drag($event, 'TestElement')"
            @dragend="dragend($event, 'TestElement')"
            class="droppable-element"

        >
          <!--          unselectable="on"-->
          <!--          draggable="true"-->

          Droppable Element (Drag me!)
        </div>
        <div
            @drag="drag($event, 'TestElement2')"
            @dragend="dragend($event, 'TestElement2')"

            class="droppable-element aa dragging"
            draggable="true"

            id="hihi"
            ref="itemSide"
        >

          <!--          unselectable="on"-->

          <!--          :style="{position: 'relative',zIndex:999,pointerEvents: 'all',cursor:'pointer'}"-->

          <p>testx</p>

          <img
              src="https://images.fpt.shop/unsafe/filters:quality(90)/fptshop.com.vn/uploads/images/tin-tuc/82409/Originals/select-default-operating-system-with-boot-camp-on-mac.png"
              width="50"
              height="50"
          />
          <div class="overlap"></div>
        </div>
      </div>
      <div class="content_right">
        <div id="content" @dragover="dragover($event)">
          <!-- Add to show rtl support -->
          <input type="checkbox" v-model="draggable"/> Draggable
          <input type="checkbox" v-model="resizable"/> Resizable
          <input type="checkbox" v-model="mirrored"/> Mirrored
          <input type="checkbox" v-model="responsive"/> Responsive
          <input type="checkbox" v-model="preventCollision"/> Prevent Collision
          <div style="margin-top: 10px; margin-bottom: 10px">
            Row Height: <input type="number" v-model="rowHeight"/> Col nums:
            <input type="number" v-model="colNum"/> Margin x:
            <input type="number" v-model="marginX"/> Margin y:
            <input type="number" v-model="marginY"/>
          </div>

          <grid-layout
              ref="gridlayout"
              :margin="[parseInt(marginX), parseInt(marginY)]"
              :layout.sync="layout"
              :col-num="parseInt(colNum)"
              :row-height="rowHeight"
              :is-draggable="draggable"
              :is-resizable="resizable"
              :is-mirrored="mirrored"
              :prevent-collision="preventCollision"
              :vertical-compact="compact"
              :use-css-transforms="true"
              :responsive="responsive"
          >
            <grid-item
                v-for="item in layout"
                :key="item.i"
                :x="item.x"
                :y="item.y"
                :w="item.w"
                :h="item.h"
                :i="item.i"
                :min-w="item.minW"
                :max-w="item.maxW"
                :max-h="item.maxH"
                :min-x="item.minX"
                :max-x="item.maxX"
                :min-y="item.minY"
                :max-y="item.maxY"
                :static="item.static"
                :preserve-aspect-ratio="item.preserveAspectRatio"
                @resize="resize"
                @removeItem="removeItem($event)"
            >
              <component
                  v-bind:is="item.componentName || 'TestElement'"
                  :text="item.i"
                  @removeItem="removeItem($event)"
              ></component>
            </grid-item>

            <div class="grid_wrap">
              <div class="grid_inner">
                <div class="test" v-for="item in 40" :key="item"></div>
              </div>
            </div>
          </grid-layout>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import GridItem from "./customs/components/GridItem.vue";
import GridLayout from "./customs/components/GridLayout.vue";
import TestElement from "./customs/components/TestElement.vue";
import TestElement2 from "./customs/components/TestElement2.vue";
import Vue from "vue";

import "@interactjs/auto-start";
import "@interactjs/actions/drag";
import "@interactjs/actions/resize";
import "@interactjs/modifiers";
import "@interactjs/dev-tools";
import interact from "@interactjs/interact";
import {
  bottom,
  compact,
  getLayoutItem,
  moveElement,
  validateLayout,
  cloneLayout,
  getAllCollisions,
} from "@/helpers/utils";
import {
  getBreakpointFromWidth,
  getColsFromBreakpoint,
  findOrGenerateResponsiveLayout,
} from "@/helpers/responsiveUtils";
//var eventBus = require('./eventBus');
import {getControlPosition, createCoreData} from "@/helpers/draggableUtils";

import {
  addWindowEventListener,
  removeWindowEventListener,
} from "@/helpers/DOM";

let mouseXY = {x: null, y: null};
let DragPos = {x: null, y: null, w: 1, h: 1, i: null};
let position = {x: 0, y: 0};
let testLayout = [
  {
    x: 0,
    y: 0,
    w: 1,
    h: 1,
    i: "0",
    resizable: true,
    draggable: true,
    static: false,
    maxH: 1,
    componentName: "TestElement2",
  },
  {
    x: 2,
    y: 0,
    w: 1,
    h: 1,
    i: "2",
    resizable: false,
    draggable: false,
    static: false,
    maxH: 1,
    maxW: 1,
  },
  {
    x: 1,
    y: 0,
    w: 1,
    h: 1,
    i: "22",
    resizable: false,
    draggable: false,
    static: false,
    maxH: 1,
    maxW: 1,
  },
  {
    x: 0,
    y: 1,
    w: 1,
    h: 1,
    i: "222",
    resizable: false,
    draggable: false,
    static: false,
    maxH: 1,
    maxW: 1,
  },
  {
    x: 0,
    y: 2,
    w: 2,
    h: 1,
    i: "3",
    resizable: false,
    draggable: false,
    static: false,
  },
  {
    x: 2,
    y: 2,
    w: 2,
    h: 1,
    i: "4",
    resizable: false,
    draggable: false,
    static: false,
  },
];

export default {
  name: "app",
  components: {
    GridLayout,
    GridItem,
    TestElement,
    TestElement2,
  },
  data() {
    return {
      elmClone:null,
      interactObj: null,
      layout: JSON.parse(JSON.stringify(testLayout)),
      draggable: true,
      resizable: true,
      mirrored: false,
      responsive: false,
      preventCollision: false,
      compact: true,
      rowHeight: 155,
      colNum: 4,
      index: 0,
      marginX: 10,
      marginY: 10,
    };
  },
  created() {

  },
  mounted: function () {
    const self = this;

    if (this.interactObj === null || this.interactObj === undefined) {
      this.interactObj = interact(this.$refs.itemSide);
      console.log(1)
      console.log("this.interactObj", this.interactObj)

      const opts = {
        ignoreFrom: null,
        allowFrom: null,
      };
      this.interactObj.draggable(opts);
      /*this.interactObj.draggable({allowFrom: '.vue-draggable-handle'});*/
      if (!this.dragEventSet) {
        this.dragEventSet = true;
        this.interactObj.on("dragstart dragmove dragend", function (event) {
          self.handleDrag(event);
        });
      }
    }
    this.index = this.layout.length;

    document.addEventListener(
        "dragover",
        function (e) {
          // console.log("addEventListener")
          mouseXY.x = e.clientX;
          mouseXY.y = e.clientY;
        },
        false
    );
  },
  methods: {
    handleDrag(event) {
      if (this.static) return;
      if (this.isResizing) return;
      const position = getControlPosition(event);
      const { x, y } = position;
      const {currentTarget, interaction} = event;
      let element = currentTarget;
      const {offsetTop, offsetLeft} = currentTarget;
      switch (event.type) {
        case "dragstart": {
          element = currentTarget.cloneNode(true);
          console.log("element",element)
          // Add absolute positioning so that cloned object lives right on top of the original object
          element.style.position = "absolute";
          element.style.left = offsetLeft;
          element.style.top = offsetTop;
          element.style.zIndex = 99;
          element.setAttribute("id","elmMoveClone")
          const container = document.querySelector(".content_wrap");
          container && container.appendChild(element);
          this.elmClone = element
          break;
        }
        case "dragend": {
          document.getElementById("elmMoveClone").remove();
          break;
        }
        case "dragmove": {
          this.elmClone.style.left = x+"px";
          this.elmClone.style.top = y+"px";
          break;
        }
      }
    },
    onDragEnd: function (event) {
      if (!event.target.drag) {
        return;
      }
      // Define persist true to let the source persist and drop the target, otherwise persist the target.
      var persist = true;
      if (persist || event.out) {
        event.target.parentNode.removeChild(event.target);
      } else {
        event.target.parentNode.removeChild(event.target.drag.source);
      }
      event.target.classList.remove("dragging");
      event.target.drag = null;
    },

    onDragOver: function (event) {
      event.preventDefault();
    },

    dragEvent: function (eventName, id, x, y, h, w) {
      console.log(6969)
      this.verticalCompact = false
      //console.log(eventName + " id=" + id + ", x=" + x + ", y=" + y);
      let l = getLayoutItem(this.layout, id);
      //GetLayoutItem sometimes returns null object
      if (l === undefined || l === null) {
        l = {x: 0, y: 0};
      }

      if (eventName === "dragmove" || eventName === "dragstart") {
        this.placeholder.i = id;
        this.placeholder.x = l.x;
        this.placeholder.y = l.y;
        this.placeholder.w = w;
        this.placeholder.h = h;
        this.$nextTick(function () {
          this.isDragging = true;
        });
        //this.$broadcast("updateWidth", this.width);
        this.eventBus.$emit("updateWidth", this.width);
      } else {
        this.verticalCompact = true

        this.$nextTick(function () {
          this.isDragging = false;
        });
      }

      // Move the element to the dragged location.
      this.layout = moveElement(
          this.layout,
          l,
          x,
          y,
          true,
          this.preventCollision,
          this.isDraggable
      );
      compact(this.layout, this.verticalCompact);
      // needed because vue can't detect changes on array element properties
      this.eventBus.$emit("compact");
      this.updateHeight();
      // if (eventName === "dragend") this.$emit("layout-updated", this.layout);
    },


    removeItem: function (i) {
      console.log("### REMOVE " + i);
      const index = this.layout.map((item) => item.i).indexOf(i);
      this.layout.splice(index, 1);
    },

    drag: function (e) {
      // e.classList.add('is-moving');
      // // e.target.classList.add('grabbing');
      // if (e.preventDefault) {
      //   e.preventDefault();
      // }
      // e.dataTransfer.dropEffect = 'move';
      // e.stopPropagation();

      let parentRect = document
          .getElementById("content")
          .getBoundingClientRect();
      let mouseInGrid = false;

      if (
          mouseXY.x > parentRect.left &&
          mouseXY.x < parentRect.right &&
          mouseXY.y > parentRect.top &&
          mouseXY.y < parentRect.bottom
      ) {
        mouseInGrid = true;
      }
      if (
          mouseInGrid === true &&
          this.layout.findIndex((item) => item.i === "drop") === -1
      ) {
        this.layout.push({
          x: (this.layout.length * 2) % (this.colNum || 12),
          y: this.layout.length + (this.colNum || 12), // puts it at the bottom
          w: 1,
          h: 1,
          i: "drop",
        });
      }
      let index = this.layout.findIndex((item) => item.i === "drop");
      if (index !== -1) {
        try {
          this.$refs.gridlayout.$children[
              this.layout.length
              ].$refs.item.style.display = "none";
        } catch (error) {
          console.log("");
        }
        let el = this.$refs.gridlayout.$children[index];
        el.dragging = {
          top: mouseXY.y - parentRect.top,
          left: mouseXY.x - parentRect.left,
        };
        let new_pos = el.calcXY(
            mouseXY.y - parentRect.top,
            mouseXY.x - parentRect.left
        );
        if (mouseInGrid === true) {
          this.$refs.gridlayout.dragEvent(
              "dragstart",
              "drop",
              new_pos.x,
              new_pos.y,
              1,
              1
          );
          DragPos.i = String(index);
          DragPos.x = this.layout[index].x;
          DragPos.y = this.layout[index].y;
        }
        if (mouseInGrid === false) {
          this.$refs.gridlayout.dragEvent(
              "dragend",
              "drop",
              new_pos.x,
              new_pos.y,
              1,
              1
          );
          this.layout = this.layout.filter((obj) => obj.i !== "drop");
        }
      }
    },
    dragover: function (e) {
      // e.target.classList.add('grabbing');
      // document.body.style.cursor = "pointer"
      // let dt =  event.dataTransfer;
      // dt.effectAllowed = "copyMove";
      // dt.setData("text/plain", "Foo");
      e.dataTransfer.dropEffect = "copy";
    },
    dragend: function (e, componentName) {
      // e.dataTransfer.dropEffect = "move";
      e.target.style.cursor = 'default';

      let parentRect = document
          .getElementById("content")
          .getBoundingClientRect();
      let mouseInGrid = false;
      if (
          mouseXY.x > parentRect.left &&
          mouseXY.x < parentRect.right &&
          mouseXY.y > parentRect.top &&
          mouseXY.y < parentRect.bottom
      ) {
        mouseInGrid = true;

      }
      if (mouseInGrid === true) {
        // alert(`ID ${id} Dropped element props:\n${JSON.stringify(DragPos, ['x', 'y', 'w', 'h'], 2)}`);
        this.$refs.gridlayout.dragEvent(
            "dragend",
            "drop",
            DragPos.x,
            DragPos.y,
            1,
            1
        );
        this.layout = this.layout.filter((obj) => obj.i !== "drop");
        // UNCOMMENT below if you want to add a grid-item
        this.layout.push({
          x: DragPos.x,
          y: DragPos.y,
          w: 1,
          h: 1,
          i: DragPos.i,
          componentName: componentName,
        });
        this.$refs.gridLayout.dragEvent(
            "dragend",
            DragPos.i,
            DragPos.x,
            DragPos.y,
            1,
            1
        );
        try {
          this.$refs.gridLayout.$children[
              this.layout.length
              ].$refs.item.style.display = "block";
        } catch (error) {
          console.log(error);
        }
      }
    },
    resize: function (i, newH, newW, newHPx, newWPx) {
      console.log(
          "RESIZE i=" +
          i +
          ", H=" +
          newH +
          ", W=" +
          newW +
          ", H(px)=" +
          newHPx +
          ", W(px)=" +
          newWPx
      );
    },
  },
};
</script>


<style lang="scss">
[draggable] {
  cursor: pointer !important;
}

[ondragstart] {
  cursor: grabbing !important;

}

[ondrag] {
  cursor: grabbing !important;

}

[ondragover] {
  cursor: grabbing !important;

}

[draggable]:enabled {
  cursor: grabbing !important;
}

[draggable]:active {
  cursor: pointer !important;
}

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}

.grid_wrap {
  padding: 10px;
  position: absolute;
  width: calc(100% - 20px);
}

.grid_inner {
  display: grid;
  grid-template-columns: auto auto auto auto;
  grid-gap: 10px;
}

.test {
  border: 1px dashed;
  height: 155px;
}

.overlap {
  position: absolute;
  width: 100%;
  height: 100%;
  left: 0;
  top: 0;
  z-index: 1;
}

.vue-grid-item {
  z-index: 2;
}

.droppable-element {
  width: 150px;
  text-align: center;
  background: #fdd;
  border: 1px solid black;
  margin: 10px 0;
  padding: 10px;
  position: relative;
}

.vue-grid-layout {
  background: #eee;
}

.layoutJSON {
  background: #ddd;
  border: 1px solid black;
  margin-top: 10px;
  padding: 10px;
}

.layoutJSON {
  background: #ddd;
  border: 1px solid black;
  margin-top: 10px;
  padding: 10px;
}

.columns {
  -moz-columns: 120px;
  -webkit-columns: 120px;
  columns: 120px;
}

.content_wrap {
  display: flex;
}

.content_flex {
  flex: 0 0 300px;
}

.content_right {
  flex: 0 0 1;
}

.aa {
  -moz-user-select: none;
  -ms-user-select: none;
  -webkit-user-select: none;
  user-select: none;
  cursor: move; /* fallback: no `url()` support or images disabled */
  cursor: -webkit-grabbing; /* Chrome 1-21, Safari 4+ */
  cursor: -moz-grabbing; /* Firefox 1.5-26 */
  cursor: grabbing; /* W3C standards syntax, should come least */
}

.aa:active {
  cursor: move; /* fallback: no `url()` support or images disabled */
  cursor: -webkit-grabbing; /* Chrome 1-21, Safari 4+ */
  cursor: -moz-grabbing; /* Firefox 1.5-26 */
  cursor: grabbing; /* W3C standards syntax, should come least */
}

.aa:disabled {
  cursor: move; /* fallback: no `url()` support or images disabled */
  cursor: -webkit-grabbing; /* Chrome 1-21, Safari 4+ */
  cursor: -moz-grabbing; /* Firefox 1.5-26 */
  cursor: grabbing; /* W3C standards syntax, should come least */
}

.aa:-moz-drag-over {
  cursor: move; /* fallback: no `url()` support or images disabled */
  cursor: -webkit-grabbing; /* Chrome 1-21, Safari 4+ */
  cursor: -moz-grabbing; /* Firefox 1.5-26 */
  cursor: grabbing; /* W3C standards syntax, should come least */
}

.is-moving {
  transform: scale(1.5);
  background: rgba(black, 0.8);
  cursor: pointer
}

.dropzone {
  width: 500px;
  height: 200px;
  background-color: silver;
}

.block {
  position: absolute;
  background-color: pink;
  margin: 10px;
  border: 20px solid pink;
}

.draggable {
  position: absolute;
  cursor: pointer; /* IE */
  cursor: -webkit-grab;
  cursor: grab;
}

.dragging {
  cursor: -webkit-grabbing;
  cursor: grabbing;
  background-color: red;
}
</style>

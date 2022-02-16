<template>
  <div>
    <div
        @drag="drag($event, 'TestElement2')"
        @dragend="dragend($event, 'TestElement2')"

        class="droppable-element aa dragging"
        draggable="true"
        @dragover="dragover($event)"
        @dragstart="dragstart($event)"
        @dragenter="dragenter($event)"
        id="hihi"
    >

      <!--          unselectable="on"-->

      <!--          :style="{position: 'relative',zIndex:999,pointerEvents: 'all',cursor:'pointer'}"-->

      <p>test</p>

      <img
          src="https://images.fpt.shop/unsafe/filters:quality(90)/fptshop.com.vn/uploads/images/tin-tuc/82409/Originals/select-default-operating-system-with-boot-camp-on-mac.png"
          width="50"
          height="50"
      />
      <div class="overlap"></div>
    </div>
  </div>
</template>

<script>
import Vue from "vue";

export default {
  name: "OutSide",
  // props: {
  //   text: {
  //     type: String,
  //     default: "x",
  //   },
  // },
  created() {
    const self = this;


    self.dragEventHandler = function (eventType, i, x, y, h, w) {
      self.dragEvent(eventType, i, x, y, h, w);
    };

    self._provided.eventBus = new Vue();
    self.eventBus = self._provided.eventBus;
    self.eventBus.$on("dragEvent", self.dragEventHandler);
  },
  data: function () {
    return {};
  },
  mounted: function () {
    console.log("### " + this.text + " ready!");
  },
};
</script>

<script type="module">
export default {
  data() {
    return {
      isDragging: false,
      mouseXPos: 0,
      mouseYPos: 0,
      x: null,
      y: null,
      tempFixed: false, // Temporary fixed positioning
      originaParent: null,
      canDrag: true,
      dropZoneCount: 6,
    };
  },
  mounted() {
    window.addEventListener("mousemove", this.handleMouseMove);
    window.addEventListener("mouseup", this.handleMouseUp);
  },
  beforeUnmount() {
    window.removeEventListener("mousemove", this.handleMouseMove);
    window.removeEventListener("mouseup", this.handleMouseUp);
  },
  methods: {
    handleMouseMove(e) {
      if (this.isDragging) {
        const diffX = e.clientX - this.mouseXPos;
        const diffY = e.clientY - this.mouseYPos;
        this.x += diffX;
        this.y += diffY;
      }
      this.mouseXPos = e.clientX;
      this.mouseYPos = e.clientY;
    },
    handleMouseUp() {
      if (!this.isDragging) return;
      this.isDragging = false;
      this.tempFixed = false; // Restore normal positioning

      // Get drop zones
      const img = this.$refs.image;
      const dropZones = this.$refs.dropZones;
      let dropped = false;
      dropZones.forEach((zone) => {
        const rect = zone.getBoundingClientRect();
        // Check if image is inside a zone
        if (
          this.mouseXPos >= rect.left &&
          this.mouseXPos <= rect.right &&
          this.mouseYPos >= rect.top &&
          this.mouseYPos <= rect.bottom
        ) {
          zone.append(img);
          this.x = 0; // Reset position relative to zone B
          this.y = 0;
          this.canDrag = false;
          dropped = true
        }        
      });
      if (!dropped) {
          this.originaParent.append(img)
          this.x = 0;
          this.y = 0;
        }
    },
    handleMouseDown(e) {
      if (!this.canDrag) return; // Prevent dragging if canDrag is false
      this.isDragging = true;
      this.tempFixed = true;
      const img = this.$refs.image;
      this.originaParent = img.parentElement;
      const imgRect = img.getBoundingClientRect();
      document.body.appendChild(img);

      // Set fixed position for smooth dragging
      img.style.position = "fixed";
      img.style.zIndex = "1000";
      this.x = imgRect.left;
      this.y = imgRect.top;
    },
  },
};
</script>

<template>
  <div class="start-zone" ref="zoneA">
    <img
      alt="logo"
      src="./assets/logo.svg"
      :style="{
        left: `${x}px`,
        top: `${y}px`,
        cursor: isDragging ? 'grabbing' : 'grab',
        position: tempFixed ? 'fixed' : 'absolute', // temporary fixed position
      }"
      draggable="false"
      ref="image"
      @mousedown="handleMouseDown"
    />
  </div>
  <div class="drop-area">
    <div v-for="index in dropZoneCount" :key="index" class="drop-zone" ref="dropZones"></div>
  </div>
</template>

<style scoped>
img {
  cursor: grab;
  position: absolute;
  height: 100px;
  width: 100px;
  z-index: 1;
}
.start-zone {
  width: 100px;
  height: 100px;
  background-color: gray;
  display: flex;
  align-items: center;
  justify-content: center;
}
.drop-area {
  position: absolute;
  display: flex;
  flex-direction: row;
  gap: 20px;
  top: 200px;
  left: 200px;
  padding: 30px;
  border: 1px solid gray;
}
.drop-zone {
  width: 100px;
  height: 100px;
  background-color: lightblue;
  position: relative;
}
</style>

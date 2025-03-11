<script type="module">
export default {
  data() {
    return {
      isDragging: false,
      mouseXPos: 0,
      mouseYPos: 0,
      x: null,
      y: null,
      currentZone: "zone-a", // Track the current zone
      tempFixed: false, // Temporary fixed positioning
      originaParent: null,
      canDrag: true,
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
      if (!this.isDragging) return
      this.isDragging = false
      this.tempFixed = false // Restore normal positioning

      // Get drop zones
      const zoneA = this.$refs.zoneA;
      const zoneB = this.$refs.zoneB;
      const img = this.$refs.image;
      const zoneBRect = zoneB.getBoundingClientRect();

      // Check if image is inside zoneB
      if (
        this.mouseXPos >= zoneBRect.left &&
        this.mouseXPos <= zoneBRect.right &&
        this.mouseYPos >= zoneBRect.top &&
        this.mouseYPos <= zoneBRect.bottom
      ) {
        this.currentZone = "zone-b"; // Switch to zone B
        zoneB.append(img)
        this.x = 0; // Reset position relative to zone B
        this.y = 0;
        this.canDrag = false;
      } else {
        zoneA.append(img)
        this.x = 0; // Reset position relative to zone A
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
      document.body.appendChild(img)

      // Set fixed position for smooth dragging
      img.style.position = "fixed";
      img.style.zIndex = "1000";
      this.x = imgRect.left;
      this.y = imgRect.top;     
    }
  },
};
</script>

<template>
  <div class="zone-a" ref="zoneA">
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
  <div class="zone-b" ref="zoneB">
  </div>
</template>

<style scoped>
div {
  z-index: 0;
  position: absolute;
  width: 100px;
  height: 100px;
  background-color: gray;
  &.zone-a {
    top: 50px;
    left: 100px;
  }
  &.zone-b {
    top: 200px;
    left: 200px;
  }
}
img {
  cursor: grab;
  position: absolute;
  height: 100px;
  width: 100px;
  z-index: 1;
}
.fixed {
  position: fixed;
}
</style>

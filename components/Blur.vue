<template>
  <div>
    <div class="warningContainer">
      <div class="warningText">Please avoid using smartphone while walking.</div>
    </div>
    <div class="blur"></div>
  </div>
</template>

<style scoped>
.warningContainer {
  position: fixed;
  width: 100%;
  height: 100%;
  z-index: 100;
  padding: 1rem;
}
.warningText {
  display: flex;
  justify-content: center;
  color: red;
  font-weight: 900;
}
.blur {
  position: fixed;
  backdrop-filter: blur(8px) brightness(20%);
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
}
</style>
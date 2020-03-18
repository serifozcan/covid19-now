<template>
  <div class="link-wrapper">
    <div class="github-link">
      <a href="https://github.com/serifozcan/covid19-now" target="_blank" rel="noopener">
        <img src="@/assets/github.svg" />
      </a>
    </div>
  </div>
</template>

<script>
export default {};
</script>

<style scoped>
.link-wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
  position: absolute;
  bottom: 40px;
  right: 0;
}
.github-link {
  border-radius: 5px 0 0 5px;
  background: #555566;
}
.github-link img {
  width: 30px;
  height: 30px;
  padding: 10px;
  transition: 0.3s all ease-in-out;
}
.github-link img:hover {
  opacity: 0.7;
}
.github-link img:active {
  transform: scale(0.9);
}
</style>

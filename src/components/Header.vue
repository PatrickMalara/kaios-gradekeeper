<template>
  <header class="header">
    <span>{{ title }}</span>
  </header>
</template>

<script>
export default {
  name: "Header",
  props: {
    title: {
      type: String,
      required: true
    }
  }
};
</script>

<style scoped>
.header {
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: indianred;
  padding: 3px 10px;
  text-align: center;
  position: fixed;
  width: 100%;
  margin-bottom: calc( 36px + 10px + 16px );
}

.header span {
  font-size: 16px;
  font-weight: 600;
  color: #ffffff;
  text-transform: uppercase;
}

</style>

<template>
    <div id="main" class="app-main">
        <router-view  v-if="isRouterAlive"></router-view>
    </div>
</template>

<script>
export default {
  data() {
    return {
       isRouterAlive: true
    };
  },
    provide () {
    return {
      reload: this.reload
    }
  },
  mounted() {},
  beforeDestroy() {},
  methods: {
       reload () {
      this.isRouterAlive = false
      this.$nextTick(function () {
        this.isRouterAlive = true
      })
    }
  }
};
</script>

<style>
html,
body {
  width: 100%;
  height: 100%;
  background: #f0f0f0;
  /* overflow: hidden; */
}

.app-main {
  width: 100%;
  height: 100%;
}

.br button {
  margin-right: 5px;
}

.operation button {
  margin-right: 5px;
}

.ivu-btn-text:focus {
  box-shadow: none !important;
}
</style>

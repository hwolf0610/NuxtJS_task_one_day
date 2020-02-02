<template>
  <div class="container">
    <div>
      <logo />
      <h1 class="title">hello_world</h1>
      <h2 class="subtitle">My well-made Nuxt.js project</h2>
      <nuxt-link :to="{ path: '/page1', query: { id: 'Page1'}}">
        Page1
      </nuxt-link><nuxt-link :to="{ path: '/page2', query: { id: 'Page2'}}">
        Page2
      </nuxt-link>
    </div>
  </div>
</template>

<script>
import Logo from "~/components/Logo.vue";

export default {
  components: {
    Logo
  }
};
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont,
    "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>

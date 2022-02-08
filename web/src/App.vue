<template>
  <div>
    <img alt="NFT Hub" id="logo" src="./assets/logo.png" />
    <EpicNFT />
  </div>
</template>

<script>
import EpicNFT from "./components/EpicNFT.vue";

export default {
  name: "App",
  components: {
    EpicNFT,
  },
};
</script>

<style>
body,
html {
    margin: 0;
}

#logo {
  width: 50%;
}

body {
    padding: 0;
    font-family: -apple-system, Inter, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen', 'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue', sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
}

code {
    font-family: source-code-pro, Menlo, Monaco, Consolas, 'Courier New', monospace;
}

#app {
    height: 100vh;
    background-color: black;
    text-align: center;
}

</style>

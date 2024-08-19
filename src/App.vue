<template>
  <ChatScreen />
</template>

<script>
import ChatScreen from "./components/ChatScreen.vue";

export default {
  name: "App",
  components: {
    ChatScreen,
  },
};
</script>

<style>
#app {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: Arial, sans-serif;
  height: 100%;
  width: 100%;
}

/*--------------------------------------------------Configuração Padrão*/
body {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: Arial, sans-serif;
  padding: 20px;
  background: url("assets/images/tech-bg.png") no-repeat center fixed;
  background-size: cover;
  height: 100vh;
  width: 100vw;
}

html {
  scroll-behavior: smooth;
}

a {
  text-decoration: none;
}

ul,
ol,
li {
  list-style: none;
}

@media (max-width: 800px) {
  body {
    padding: 0px;
  }
}
</style>

<template>
  <div class="background-video-container">
    <video autoplay muted loop id="background-video">
      <source src="@/assets/video/matrix-rain.mp4" type="video/mp4" />
      Seu navegador não suporta a reprodução de vídeo.
    </video>
  </div>
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
/* Estilo para o vídeo de fundo */
.background-video-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  z-index: -1; /* Coloca o vídeo atrás de todo o conteúdo */
}

#background-video {
  width: 100%;
  height: 100%;
  object-fit: cover; 
}
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

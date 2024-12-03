<template>
  <div class="chat-container">
    <div class="chat-messages" id="chat-messages">
      <div
        v-for="(message, index) in messages"
        :key="index"
        :class="message.type"
      >
        {{ message.text }}
      </div>
    </div>
    <form id="chat-form" @submit.prevent="handleSubmit">
      <input
        type="text"
        v-model="userInput"
        placeholder="Digite sua mensagem..."
      />
      <button type="submit">Enviar</button>
    </form>
  </div>
</template>

<script defer type="module">
import { GoogleGenerativeAI } from "@google/generative-ai";

export default {
  name: "ChatScreen",
  data() {
    return {
      userInput: "",
      messages: [
        {
          type: "bot-message",
          text: `Bem-vindo ao Chat TI Ajuda. Este sistema foi desenvolvido por Luiz Henrique Zavatini Feltrin.`,
        },
        {
          type: "bot-message",
          text: `Olá, usuário. Sou o Chat TI Ajuda. A sua mensagem foi recebida através do protocolo TCP, porta 80, e interpretada pelo meu parser. O seu pedido foi analisado e, após uma análise semântica, concluí que você deseja iniciar uma sessão de comunicação. Para otimizar a transferência de dados e garantir uma latência mínima, me informe qual o seu objetivo principal, por favor. A minha arquitetura de rede está pronta para receber sua requisição, mas precisarei de informações adicionais para direcioná-la para o processamento adequado.`,
        },
      ],
      genAI: null,
      chatSession: null,
      conversation_id: null,
      history: [], // Histórico incremental da conversa
    };
  },
  async mounted() {
    try {
      // Captura o IP do usuário usando a API Ipify
      const ipResponse = await fetch("https://api.ipify.org?format=json");
      const ipData = await ipResponse.json();
      const user_ip = ipData.ip; // Obtendo o IP

      // Captura a localização usando o IP
      const locationResponse = await fetch(`https://ipinfo.io/${user_ip}/json`);
      const locationData = await locationResponse.json();

      // Prepara os dados a serem enviados ao servidor
      const locationInfo = {
        user_ip,
        city: locationData.city,
        region: locationData.region,
        country: locationData.country,
      };

      console.log(locationInfo);
      // Inicia uma nova conversa e obtém o ID
      const response = await fetch(
        "https://vuejs-api-gemine-chatbot-server.onrender.com/startConversation",
        // "http://localhost:5000/startConversation",
        {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify(locationInfo), // Enviando o IP ao servidor
        }
      );
      const data = await response.json();
      console.log(data);

      if (data.message == "Conversa já existe para este IP.") {
        const response = await fetch(
          `https://vuejs-api-gemine-chatbot-server.onrender.com/getConversation?user_ip=${user_ip}`
          // `http://localhost:5000/getConversation?user_ip=${user_ip}`
        );

        if (response.ok) {
          const messages = await response.json();
          // Adiciona as mensagens ao array de mensagens no front-end
          const previousMessages = messages.map((msg) => ({
            type: msg.sender === "user" ? "user-message" : "bot-message",
            text: msg.message,
          }));
          // Mantenha as duas mensagens padrão e adicione as anteriores
          this.messages = [
            ...this.messages, // Mensagens padrão
            ...previousMessages, // Mensagens anteriores
          ];
        } else {
          console.log("No previous conversation found or an error occurred.");
        }
      }
      this.conversation_id = user_ip;

      // Continue com o restante da inicialização...
      this.genAI = new GoogleGenerativeAI(
        "AIzaSyDxXkVq8_y3OsnW0_BcjEBqMv0jo_0g4Kk"
      );
      const model = this.genAI.getGenerativeModel({
        model: "gemini-1.5-flash",
      });

      this.chatSession = model.startChat({
        generationConfig: {
          temperature: 1,
          topP: 0.95,
          topK: 64,
          responseMimeType: "text/plain",
        },
        history: [
          {
            role: "user",
            parts: [
              {
                text: "Você agora é um mestre em informática e em computadores, tire todas as dúvidas do usuário e recepcione ele bem. Não precisa se apresentar, fala em uma linguagem absurdamente técnica, um nivel quase incompreenssivel de técnico, que o usuario não entenda quase nada.",
              },
            ],
          },
        ],
      });

      console.log("Chat session started!");
    } catch (error) {
      console.error("Error during initialization:", error);
    }
  },
  methods: {
    // Função que remove todos os asteriscos da resposta do bot
    removeAsterisks(text) {
      return text.replace(/\*/g, "");
    },

    async sendMessageToBot(message) {
      try {
        // Adiciona a mensagem do usuário ao histórico
        this.history.push({
          role: "user",
          parts: [{ text: message }],
        });

        console.log("Histórico enviado:", this.history);

        const result = await this.chatSession.sendMessage(message);

        const responseText = await result.response.text();
        // Remove todos os asteriscos da resposta do bot
        return this.removeAsterisks(responseText);
      } catch (error) {
        console.error("Error sending message to bot:", error);
        return "Sorry, there was an error processing your message.";
      }
    },

    appendMessage(type, message) {
      this.messages.push({ type, text: message });
    },

    async handleSubmit() {
      const userMessage = this.userInput.trim();
      if (userMessage === "") return;

      this.appendMessage("user-message", userMessage);
      this.userInput = "";

      if (!this.chatSession) {
        console.error("Chat session is not initialized");
        this.appendMessage(
          "bot-message",
          "A sessão do chat não foi inicializada corretamente."
        );
        return;
      }

      const botResponse = await this.sendMessageToBot(userMessage);
      this.appendMessage("bot-message", botResponse);

      // Preparar as mensagens para salvar
      const messages = [
        { sender: "user", message: userMessage, timestamp: new Date() },
        { sender: "bot", message: botResponse, timestamp: new Date() },
      ];

      // Enviar as mensagens ao servidor para salvar
      await fetch(
        "https://vuejs-api-gemine-chatbot-server.onrender.com/saveConversation",
        // "http://localhost:5000/saveConversation",
        {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            user_ip: this.conversation_id, // Alterado para usar user_ip
            messages,
          }),
        }
      );
    },
  },
};
</script>
<style scoped>
/*--------------------------------------------------Fontes*/
@import url("https://fonts.googleapis.com/css2?family=Raleway:wght@300&display=swap");
@import url("https://fonts.googleapis.com/css2?family=Raleway:wght@100&display=swap");

/*--------------------------------------------------Scroll bar*/
.chat-messages::-webkit-scrollbar {
  width: 3px;
}

.chat-messages::-webkit-scrollbar-track {
  background: rgb(59, 59, 59);
}

.chat-messages::-webkit-scrollbar-thumb {
  background-color: rgba(255, 255, 255, 0);
  border-radius: 1px;
  border: 10px solid #ffffff00;
}

.chat-container {
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  max-width: 600px;
  margin: 0 auto;
  border-radius: 8px;
  overflow: hidden;
  background: rgba(0, 0, 0, 0.4);
  box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
  backdrop-filter: blur(7.5px);
  -webkit-backdrop-filter: blur(7.5px);
  border-radius: 10px;
  box-shadow: 0 4px 8px #0000001a;
}

.chat-messages {
  padding: 10px;
  overflow-y: scroll;
}

.user-message {
  background-color: #4caf50;
  padding: 8px;
  border-radius: 8px;
  margin-bottom: 8px;
  color: white;
}

.bot-message {
  background-color: #404040;
  color: white;
  padding: 8px;
  border-radius: 8px;
  margin-bottom: 8px;
  font-size: 15px;
}

form {
  display: flex;
  padding: 10px;
  background-color: #333;
  align-items: center;
  font-size: 15px;
  background: rgba(0, 0, 0, 0.3);
  box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
  backdrop-filter: blur(7.5px);
  -webkit-backdrop-filter: blur(7.5px);
}

input[type="text"] {
  flex: 1;
  padding: 8px;
  border: 0px solid #555;
  border-radius: 20px;
  outline: none;
  color: #ccc;
  background-color: #444;
  font-size: 15px;
  background: rgba(0, 0, 0, 0.3);
  box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
  backdrop-filter: blur(7.5px);
  -webkit-backdrop-filter: blur(7.5px);
}

input[type="text"]::placeholder {
  color: #c6c6c6;
}

button {
  padding: 8px 16px;
  margin-left: 10px;
  border: none;
  border-radius: 20px;
  background-color: #4caf50;
  color: white;
  cursor: pointer;
  transition: background-color 0.3s ease;
  font-size: 15px;
  -webkit-transition: background-color 0.3s ease;
  -moz-transition: background-color 0.3s ease;
  -ms-transition: background-color 0.3s ease;
  -o-transition: background-color 0.3s ease;
}

button:hover {
  background-color: #45a049;
}

@media (max-width: 800px) {
  .chat-container {
    max-width: none;
    border-radius: 0px;
  }
}
</style>

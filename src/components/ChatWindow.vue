<template>
  <div class="chat-container">
    <!-- Channel Selector -->
    <div class="channel-selector">
      <button v-for="channel in channels" :key="channel" @click="selectChannel(channel)"
        :class="{ active: channel === selectedChannel }">
        # {{ channel }}
      </button>
    </div>

    <!-- Messages Area -->
    <div class="messages">
      <div v-for="(msg, index) in messages" :key="index"
        :class="['message', msg.sender === userId ? 'sent' : 'received']">
        <div v-if="msg.sender !== userId" class="avatar">👤</div>
        <div class="message-content">
          user({{ msg.sender }}): {{ msg.message }}
        </div>
      </div>
    </div>

    <!-- Input Area -->
    <div class="input-area">
      <input v-model="newMessage" @keyup.enter="sendMessage" placeholder="Type a message..."
        :disabled="!selectedChannel" />
      <button @click="sendMessage" :disabled="!selectedChannel">➤</button>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted } from "vue";
import { io } from "socket.io-client";

export default defineComponent({
  name: "ChatWindow",
  setup() {
    const socket = io("http://localhost:8080");

    // State
    const channels = ref(["general", "random", "help"]); // Predefined channels
    const selectedChannel = ref<string | null>(null);
    const messages = ref<{ message: string; sender: string }[]>([]);
    const newMessage = ref<string>("");
    const userId = ref<string>("");

    // Join the selected channel
    const selectChannel = (channel: string) => {
      selectedChannel.value = channel;
      messages.value = []; // Clear messages when switching channels
      socket.emit("join-channel", channel);
    };

    // Listen for messages from the server
    onMounted(() => {
      userId.value = socket.id ?? ""; // Assign current socket ID
      socket.on("chat-message", (data: { message: string; sender: string }) => {
        messages.value.push(data); // Add the new message to the chat
      });
    });

    // Send a new message to the selected channel
    const sendMessage = () => {
      if (newMessage.value.trim() && selectedChannel.value) {
        socket.emit("chat-message", {
          channel: selectedChannel.value,
          message: newMessage.value,
        });
        newMessage.value = ""; // Clear the input field
      }
    };

    return {
      channels,
      selectedChannel,
      messages,
      newMessage,
      sendMessage,
      selectChannel,
      userId,
    };
  },
});
</script>

<style scoped>
/* Chat container */
.chat-container {
  display: flex;
  flex-direction: column;
  width: 500px;
  height: 700px;
  border: 1px solid #ccc;
  border-radius: 10px;
  background-color: #f9f9f9;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  font-family: Arial, sans-serif;
}

/* Channel selector */
.channel-selector {
  display: flex;
  flex-direction: column;
  padding: 10px;
  background-color: #007bff;
  color: white;
  border-top-left-radius: 10px;
  border-top-right-radius: 10px;
}

.channel-selector button {
  background: none;
  border: none;
  color: white;
  padding: 10px;
  font-size: 14px;
  cursor: pointer;
  text-align: left;
}

.channel-selector button.active {
  background-color: #0056b3;
  border-radius: 5px;
}

/* Messages area */
.messages {
  flex: 1;
  padding: 10px;
  overflow-y: auto;
  background-color: #f9f9f9;
}

.message {
  display: flex;
  align-items: flex-start;
  margin-bottom: 10px;
}

.message-content {
  max-width: 70%;
  padding: 10px;
  border-radius: 10px;
  font-size: 14px;
  line-height: 1.4;
}

.sent .message-content {
  background-color: #d1e7dd;
  margin-left: auto;
}

.received .message-content {
  background-color: #f8d7da;
}

.avatar {
  width: 32px;
  height: 32px;
  margin-right: 10px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 24px;
}

/* Input area */
.input-area {
  display: flex;
  padding: 10px;
  border-top: 1px solid #ccc;
  background-color: #fff;
}

input {
  flex: 1;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

button {
  padding: 10px;
  margin-left: 10px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}
</style>

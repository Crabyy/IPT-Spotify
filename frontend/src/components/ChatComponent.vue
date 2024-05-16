<template>
  <div class="flex items-center justify-between text-white font-bold mb-4 p-2 mx-10 rounded-lg">
    <div class="text-center ml-10">Chat</div>
  </div>

  <div class="fixed bottom-0 justify-center">
    <div class="flex flex-col items-center w-full">
      <input v-model="messageInput" @keyup.enter="sendMessage" type="text" placeholder="Enter a message" class="rounded-md text-center border border-white mb-5 h-8 text-black w-56">
      <button @click="sendMessage" class="bg-zinc-700 hover:bg-zinc-600 text-white text-md font-semibold rounded-full h-8 text-center mb-4 w-24">Send</button>
    </div>
  </div>

  <!-- Display messages -->
  <ul id="messages" class="mt-8 mx-2">
    <li v-for="msg in messages" :key="msg.id" :class="{ 'text-right': msg.sender === username, 'text-left': msg.sender !== username, 'text-white': msg.sender === username, 'text-white-600': msg.sender !== username, '': true }">

      <!-- Use conditional rendering to style sent messages -->
      <div v-if="msg.sender === username" class="bg-gray-500 shadow-sm shadow-white rounded-lg py-2 px-4 text-center mb-2 inline-block">{{ msg.text }}</div>

      <!-- Use conditional rendering to style received messages -->
      <div v-else class="bg-gray-800 shadow-sm shadow-white rounded-lg py-2 px-4 text-center mb-2 inline-block">{{ msg.sender && msg.sender !== username ? msg.sender + ': ' : '' }}{{ msg.text }}</div>
    </li>
  </ul>
</template>

 
 <script>
 import { onMounted, ref } from 'vue'; // Import ref and onMounted from Vue 3
 import io from 'socket.io-client';
 
 export default {
   setup() {
     const messageInput = ref(''); // Reactive variable for message input
     const messages = ref([]); // Reactive variable to store messages
     const socket = io('http://localhost:3000'); // Connect to your server
     const username = ref(''); // Reactive variable to store the username
 
     // Add authentication token logic
     const token = localStorage.getItem('token');
     if (token) {
       socket.emit('authenticate', token);
     }
 
     onMounted(() => {
       // Extract username from JWT token payload
       try {
         const token = localStorage.getItem('token');
         const decodedToken = JSON.parse(atob(token.split('.')[1])); // Decodes the payload part of the JWT token
         if (decodedToken) {
           username.value = decodedToken.username;
         }
       } catch (error) {
         console.error('Error fetching username from token:', error);
       }
 
       socket.on('connect', () => {
         console.log('Connected to server');
       });
 
       socket.on('chat message', (msg) => {
         messages.value.push({ id: Date.now(), text: msg.text, sender: msg.sender, isTemplate: msg.text.includes('<template>') });
         window.scrollTo(0, document.body.scrollHeight);
       });
     });
 
     const sendMessage = () => {
       if (messageInput.value.trim() !== '') {
         socket.emit('chat message', { text: messageInput.value.trim(), sender: username.value }); // Use the extracted username here
         messageInput.value = '';
       }
     };
 
     return { messageInput, sendMessage, messages, username };
   },
 };
 </script>
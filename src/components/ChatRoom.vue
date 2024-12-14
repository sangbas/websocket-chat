<template>
    <div class="container">
        <h1>Sangbas chat - <span class="connection_ready" v-if="connectionReady">Connection ready!</span></h1>
        
        <div class="messages" id="messages">
          <div class="message-container">
            <h1 class="error" v-if="connectionError"> Connection error! </h1>
            <div v-for="(m,idx) in messages" :key="'m-' + idx" style="clear:both">
              <div :class="{ 'msg-from-me' : m.from=='me', 'msg-from-other' : m.from=='other'}">
                {{m.message}}
              </div>
            </div> 
          </div>
        </div>
  
        <div class="send-zone">
          <input v-model="newMessage" type="text" placeholder="Type a message" @keyup.enter="sendMessage"/>
        </div>
    </div>
  </template>
  
  <script lang="ts">
  import { defineComponent } from "vue";
  export default defineComponent({
    name: 'ChatRoom',
    data(){
      return {
        connectionReady: false,
        connectionError: false,
        nickname: "",
        websocket: null as WebSocket | null,
        newMessage: "",
        messages: [] as any
      }
    },
    methods:{
      initChat(): void {
        //ask for a nickname
        if(this.nickname == "") this.nickname = prompt("Enter a nickname:") || "";
        var websocketUrl = process.env.VUE_APP_WEBSOCKET_URL;
        this.websocket = new WebSocket(websocketUrl);
        this.websocket.onopen    = this.onSocketOpen;
        this.websocket.onmessage = this.onSocketMessage;
        this.websocket.onerror   = this.onSockerError;
      },
      onSocketOpen(){
        this.connectionReady = true;
      },
      onSocketMessage(evt: any){
        //we parse the json that we receive
        var received = JSON.parse(evt.data);
        //check if it's our message or from a friend
        if(this.nickname != received.from) {
          this.messages.push( { from: "other", message: received.message } );
          //scroll to the bottom of the messages div
          const messages_div = document.getElementById('messages');
          messages_div!.scrollTo({top: messages_div!.scrollHeight, behavior: 'smooth'});
        }
      },
  
      onSockerError(){
        this.connectionError = true;
      },
  
      sendMessage() {
        var toSend = { from: this.nickname, message: this.newMessage };
        this.websocket!.send( JSON.stringify(toSend) );
        this.messages.push( { from: "me", message: this.newMessage } );
        this.newMessage = "";
      }
    },
    mounted() {
      this.initChat();
    }
  })
  </script>
  
  <style lang="less">
    body{
      background: #111b21;
    }
    .container{
      display: flex;
      flex-direction: column;
      margin: 0 auto;
      max-width: 768px;
      min-height: 98vh;
      position: relative;
  
      h1{
        padding: 0px;
        height: 20px;
        color: white;
        font-size: 20px;
        text-transform: uppercase;
  
        .connection_ready{
          color: greenyellow;
        }
      }
  
      .messages{
        height: 80vh;
        overflow-y: scroll;
        background: url(@/assets/background.png) no-repeat center;
        background-size: cover;
        
        
  
        .msg-from-me {
          border-radius: 7.5px;
          max-width: 65%;
          font-size: 16px;
          line-height: 19px;
          color: #e9edef;
          background: fade( #046a62, 90%);
          padding: 5px;
          margin: 20px 20px 5px 0px;
          
          float: right;
        }
        .msg-from-other {
          border-radius: 7.5px;
          max-width: 65%;
          font-size: 16px;
          line-height: 19px;
          color: #e9edef;
          background: fade( #202c33, 90%);
          padding: 5px;
          margin: 20px 0px 5px 20px;
          float: left;
        }
      }
      .send-zone{
        height: 62px;
        background: #202c33;
        display: flex;
  
        input[type='text']{
          padding: 9px 12px 11px;
          margin: 5px 10px;
          border: 1px solid #2a3942;
          background: #2a3942;
          border-radius: 8px;
          font-size: 15px;
          flex-grow: 1;
          color: white;
        }
  
      }
    }
    
  </style>
  
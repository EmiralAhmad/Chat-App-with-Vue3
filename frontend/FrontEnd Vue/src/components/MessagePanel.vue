<template>
  <div class="box">
    <div class="header-chatbox">
      <status-icon :connected="user.connected" />{{ user.username }}
    </div>
    <div class="main-chatbox">
    <div class="chatbox-container">
    <div class="chat-container">
    <div class="chat-item-box">
      <li
        v-for="(message, index) in user.messages"
        :key="index"
        :class="
        message.fromSelf
          ? 'chat-item-container'
          : 'chat-item-container chat-item-container-others'
          "
      >
        <div 
          v-if="bdelete && message.fromSelf" 
          class="chat-action"
          @click="deleteCHAT(message, index)"
          >
        <button
        @click="bdelete = false"
        type="button"
        class="btn btn-lg"
        >
        &#128465;
      </button>
      </div>
        
      <div
      v-if="!message.sent && message.fromSelf"
      class="chat-action"
      >
      <button
        type="button"
        class="btn btn-warning"
        @click="resendCHAT(message)"
      >
      &#8635;
      </button>
      </div>

       <div
          :class="
          message.fromSelf ? 'chat-item' : 'chat-item chat-item-others'
          "
          @click="bdelete = !bdelete"
          >
          <div>
          {{ message.content }}
          </div>
          <div>
          <sub>{{ message.date }}</sub>
          </div>
          </div>
          <br />
      </li>
      </div>
    </div>
    </div>
  </div>
      <form @submit.prevent="onSubmit" class="chat-form">
      <textarea 
      v-model="input" 
      placeholder="Please input your message..." 
      @keypress.enter="onSubmit"
      class="chat-input" 
      type="text"
      />
      <button :disabled="!isValid" class="send-button">Send</button>
    </form>
  </div>
</template>

<script>
import { useChatStore } from "../stores/chat"
import StatusIcon from "./StatusIcon.vue";

export default {
  name: "MessagePanel",
  emits: ["input", "deleteChat", "resendChat"],
  setup() {
    const Chat = useChatStore();
    return { Chat }
  },
  components: {
    StatusIcon,
  },
  props: {
    user: Object,
  },
  data() {
    return {
      input: "",
      bdelete: false,
    };
  },
  methods: {
    onSubmit() {
      this.$emit("input", this.input);
      this.input = "";
    },
    deleteCHAT(message, index) {
      this.$emit("resendChat", message, index);
    },
    resendCHAT(message) {
      this.$emit("resendChat", message)
    },
    displaySender(message, index) {
      return (
        index === 0 ||
        this.user.messages[index - 1].fromSelf !==
          this.user.messages[index].fromSelf
      );
    },
  },
  computed: {
    isValid() {
      return this.input.length > 0;
    },
  },
};
</script>

<style scoped>

.messages {
  margin: 0;
  padding: 20px;
}


.message {
  list-style: none;
  overflow: auto;
}

.sender {
  font-weight: bold;
  margin-top: 5px;
}

.form {
  padding: 10px;
}

.receiver {
  overflow-wrap: break-word;
  font-weight: bold;
  margin-top: 5px;
  text-align: left;
  padding: 10px;
  background: rgb(211, 211, 211);
  border-radius: 0px 15px 15px 15px;
  float: left;
  max-width: 70%;
}

.send-button {
  vertical-align: top;
  padding: 10px;
  background-color: #3D8361;
  border-radius: 50px;
  color: #fff;
  height: 60px;
  width: auto;
}

</style>

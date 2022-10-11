<template>
  <div class="container-chatbox">
    <main class="chat-box">
    <div class="left-panel">
      <div class="title-panel">Contacts</div>
      <user
        v-for="user in users"
        :key="user.userID"
        :user="user"
        :selected="selectedUser === user"
        @select="onSelectUser(user)"
      />
    </div>
    <div class="content-chatbox" v-if="selectedUser">
    <message-panel
      :user="selectedUser"
      @input="onMessage"
      @deleteChat="deleteChat"
      @resendChat="resendChat"
      class="content-chatbox"
    />
    </div>
      <div v-else class="select-message-chatbox-txt">
        Select contact to start a message
      </div>
      <div class="footer">
        <button class="btn-logout" @click="logOUT">LOGOUT</button>
      </div>
    </main> 
  </div>
</template>

<script>
import {useChatStore} from "../stores/chat"
import socket from "../socket";
import User from "./User.vue";
import MessagePanel from "./MessagePanel.vue";

export default {
  name: "Chat",
  emits: ["logOut"],
  setup() {
    const Chat = useChatStore()
    return { Chat };
  },
  components: { User, MessagePanel },
  data() {
    return {
      selectedUser: null,
      users: [],
      count: 0,
    };
  },
  methods: {
    onMessage(content) {
      const id = Date.now();
      const date = new Date().toLocaleTimeString(["it-IT"], {
        hour: "2-digit",
        minute: "2-digit",
      });
      const to = this.selectedUser.userID;
      if (this.selectedUser) {
        if (socket.connected) {
          this.Chat.addChat(id, content, date, to);
          this.selectedUser.messages.push({
            id,
            content: this.content.data,
            date,
            fromSelf: true,
            sent: true,
          });
        } else {
           this.selectedUser.messages.push({
            id,
            content: this.content.data,
            date,
            fromSelf: true,
            sent: false,
          });
        }
      }
    },
    onSelectUser(user) {
      this.selectedUser = user;
      user.hasNewMessages = false;
    },
  },
  logOUT() {
    socket.on("user disconnected", (id) => {
      for (let i = 0; i < this.users.length; i++) {
        const user = this.users[i];
        if (user.userID === id) {
          user.connected = false;
          break;
        }
      }
    });
     this.$emit("logOut")
  },
  deleteChat(messageDelete, messageIndex) {
    const to = this.selectedUser.userID;
    const messageDeleteid = messageDelete.id;
    if (this.selectedUser) {
      this.Chat.removeChat(messageDeleteid, to, messageIndex);
      this.selectedUser.messages = this.selectedUser.messages.filter(
        (item) => {
          if (item.id != messageDeleteid) {
            return item;
          }
        }
      )
    }
  },
  resendChat(messageResend) {
      const id = messageResend.id;
      const content = messageResend.content;
      const date = messageResend.date;
      const to = this.selectedUser.userID;
      if (this.selectedUser) {
        if (socket.connected) {
          console.log("connection on");
          this.Chat.addChat(id, content, date, to);
          this.selectedUser.messages.map((item) => {
            if (item.id == id) {
              item.sent = true;
            }
            return item;
          });
        } else {
          console.log("connection failed");
          this.selectedUser.messages.map((item) => {
            if (item.id == id) {
              item.sent = false;
            }
            return item;
          });
        }
      }
    },
  created() {
    socket.on("connect", () => {
      this.users.forEach((user) => {
        if (user.self) {
          user.connected = true;
        }
      });
    });

    socket.on("disconnect", () => {
      this.users.forEach((user) => {
        if (user.self) {
          user.connected = false;
        }
      });
    });

    const initReactiveProperties = (user) => {
      user.hasNewMessages = false;
    };

    socket.on("users", (users) => {
      users.forEach((user) => {
        user.messages.forEach((message) => {
          message.fromSelf = message.from === socket.userID;
        });
        for (let i = 0; i < this.users.length; i++) {
          const existingUser = this.users[i];
          if (existingUser.userID === user.userID) {
            existingUser.connected = user.connected;
            existingUser.messages = user.messages;
            return;
          }
        }
        user.self = user.userID === socket.userID;
        initReactiveProperties(user);
        this.users.push(user);
      });
      // put the current user first, and sort by username
      this.users.sort((a, b) => {
        if (a.self) return -1;
        if (b.self) return 1;
        if (a.username < b.username) return -1;
        return a.username > b.username ? 1 : 0;
      });
    });

    socket.on("user connected", (user) => {
      for (let i = 0; i < this.users.length; i++) {
        const existingUser = this.users[i];
        if (existingUser.userID === user.userID) {
          existingUser.connected = true;
          return;
        }
      }
      initReactiveProperties(user);
      this.users.push(user);
    });

    socket.on("user disconnected", (id) => {
      for (let i = 0; i < this.users.length; i++) {
        const user = this.users[i];
        if (user.userID === id) {
          user.connected = false;
          break;
        }
      }
    });

    socket.on("private message", ({ id, content, date, from, to, sent }) => {
      for (let i = 0; i < this.users.length; i++) {
        const user = this.users[i];
        const fromSelf = socket.userID === from;
        if (user.userID === (fromSelf ? to : from)) {
          user.messages.push({
            id,
            content,
            date,
            fromSelf,
            sent,
          });
          if (user !== this.selectedUser) {
            user.hasNewMessages = true;
          }
          break;
        }
      }
    });
  
   socket.on("delete message", ({ to, from, messageIndex }) => {
      for (let i = 0; i < this.users.length; i++) {
        const user = this.users[i];
        const fromSelf = socket.userID === from;
        if (user.userID === (fromSelf ? to : from)) {
          user.messages = user.messages.filter((item, index) => {
            if (index != messageIndex) {
              return item;
            }
          });
          if (user !== this.selectedUser) {
            user.hasNewMessages = true;
          }
          break;
        }
      }
    });
  },
  unmounted() {
    socket.off("connect");
    socket.off("disconnect");
    socket.off("users");
    socket.off("user connected");
    socket.off("user disconnected");
    socket.off("private message");
    socket.off("delete message");
  },
};
</script>

<style scoped>


.footer {
  position: fixed;
  left: 70px;
  bottom: 20px;
  padding: 5px;
  width: 5%;
}

.right-panel {
  width: 100%;
  max-width: 1000px;
}

.btn-logout {
  cursor: pointer;
  background: transparent;
  padding: 10px 15px 10px;
  font-weight: bold;
  color: black;
  border-radius: 5px;
  outline: none;
  margin: auto;
}
</style>
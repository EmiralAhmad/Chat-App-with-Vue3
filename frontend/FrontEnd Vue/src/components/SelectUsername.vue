<template>
  <div class="select-username">
    <div class="login-box">
    <form class="form-login" @submit.prevent="onSubmit">
      <header class="header-login">LOGIN</header>
      <input 
      class="input-login"
      type="text"
      v-model="username" 
      autocomplete="off"
      placeholder="USERNAME" 
      />
      <button :disabled="!isValid" class="btn-login">LOGIN</button>
    </form>
  </div>
  </div>
</template>

<script>
import { ref } from "vue";

export default {
  name: "SelectUsername",
  emits: ["input"],
  data() {
    return {
      username: ref(""),
    };
  },
  computed: {
    isValid() {
      return this.username.length > 2;
    },
  },
  methods: {
    onSubmit() {
      this.$emit("input", this.username); 
      this.username = "";
    },
  },
};
</script>

<style scoped>


.select-username {
  display: flex;
  width: 100%;
  height: 100vh;
  background-color: whitesmoke;
}

.login-box {
  background-color: #CEE5D0;
  box-shadow: rgba(0, 0, 0, 0.35) 0px 5px 25px;
  border-radius: 15px;
  width: 100%;
  box-sizing: border-box;
  padding: 20px;
  max-width: 450px;
  margin: auto;
}

.header-login {
  padding: 30px;
  text-align: center;
  font-weight: bold;
  color: #1C6758;
  font-size: 36px;
  border-bottom: 1px solid rgb(223, 223, 223);
}

.form-login {
  display: flex;
  flex-direction: column;
  margin-top: 25px;
}

.input-login {
  color: black;
  margin-bottom: 10px;
  border: none;
  outline: none;
  font-size: 12px;
  padding: 10px;
  border-radius: 5px;
  box-shadow: rgba(67, 71, 85, 0.27) 0px 0px 0.25em,
    rgba(90, 125, 188, 0.05) 0px 0.25em 1em;
}

.btn-login {
  width: 100%;
  max-width: 415px;
  margin: auto;
  border: none;
  padding: 15px;
  cursor: pointer;
  border-radius: 5px;
  font-weight: bold;
  color: white;
  outline: none;
  background: #7FB77E;
  margin-top: 5px;
  margin-bottom: 30px;
}
</style>

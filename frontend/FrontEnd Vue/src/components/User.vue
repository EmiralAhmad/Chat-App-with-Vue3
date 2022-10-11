<template>
  <div class="user" @click="onClick" :class="{ selected: selected }">
    <div class="description">
      <div class="name" v-if="!user.self">
        {{ user.username }}
      </div>
      <div class="status" v-if="!user.self">
        <status-icon :connected="user.connected" />{{ status }}
      </div>
    </div>
    <div v-if="user.hasNewMessages" class="new-messages">!</div>
  </div>
</template>

<script>
import StatusIcon from "./StatusIcon.vue";

export default {
  name: "ComponentUser",
  emits: ["select"],
  components: { StatusIcon },
  data() {
    return {
      count: 0,
    }
  },
  props: {
    user: Object,
    selected: Boolean,
  },
  methods: {
    onClick() {
      this.$emit("select");
    },
  },
  computed: {
    status() {
      return this.user.connected ? "online" : "offline";
    },
  },
};
</script>

<style scoped>

.selected {
  background-color: #B1D7B4;
  color: white
}

.user {
  padding: 10px;
}

.description {
  display: inline-block;
}

.status {
  color: black;
}

.new-messages {
  color: white;
  background-color: red;
  width: 20px;
  border-radius: 5px;
  text-align: center;
  float: right;
  margin-top: 10px;
}
</style>

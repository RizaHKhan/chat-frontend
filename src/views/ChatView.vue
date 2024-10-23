<template>
  <div class="chat">
    <div class="chat__messages">
      <div v-for="(chat, index) in chats" :key="index" class="chat__messages--message">
        <span class="message" :class="[
          currentUser === chat.user
            ? 'chat__messages--message--you'
            : 'chat__messages--message--other',
        ]">{{ chat.message }}</span>
      </div>
    </div>

    <input ref="input" type="text" placeholder="Enter text here" class="chat__input" @keypress.enter="submitMessage" />
  </div>
</template>

<script setup lang="ts">
import { ref, type Ref, onMounted } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const user = route.params.user
const input: Ref<HTMLFormElement | null> = ref(null)
const socket: Ref<WebSocket | null> = ref(null)
const currentUser = ref('khanr')
const chats = ref([
  { user, message: 'Hello' },
  { user: 'other', message: 'Hi' },
  { user, message: 'How is it going?' },
  { user: 'other', message: 'Good, you?' },
])

const submitMessage = (event: KeyboardEvent) => {
  event.preventDefault()
  const { value: message } = event.target as HTMLFormElement
  chats.value.push({ user: currentUser.value, message })

  if (socket?.value?.readyState === WebSocket.OPEN) {
    console.log('message send to socket')
    socket.value.send(
      JSON.stringify({
        action: 'message',
        message: { user: currentUser.value, message },
      }),
    )
  }

  if (input?.value?.value) {
    input.value.value = ''
  }
}

const connectHandler = () => {
  console.log('connected')
}

const disconnectHandler = () => {
  console.log('disconnected')
}

const messageHandler = (event: MessageEvent) => {
  const receivedMessage = JSON.parse(event.data)
  chats.value.push(receivedMessage)
}

const errorHandler = (e: Event) => {
  console.log('e', e)
}

onMounted(() => {
  socket.value = new WebSocket(
    'wss://6s0eopwa31.execute-api.us-east-1.amazonaws.com/dev',
  )

  socket.value.onopen = connectHandler
  socket.value.onmessage = messageHandler
  socket.value.onerror = errorHandler
  socket.value.onclose = disconnectHandler
})
</script>

<style lang="scss" scoped>
.chat {
  display: flex;
  flex-direction: column;
  grid-gap: 25px;
  width: 100%;
  height: 100%;

  &__messages {
    border: solid 1px var(--color-border);
    border-radius: 10px;
    padding: 10px;
    display: flex;
    flex-direction: column;
    grid-gap: 5px;

    &--message {
      display: flex;
      font-size: 18px;

      .message {
        border-radius: 20px;
        padding: 5px 25px;
      }

      &--you {
        background-color: green;
      }

      &--other {
        margin-left: auto;
        background-color: grey;
      }
    }
  }

  &__input {
    width: 100%;
    font-size: 20px;
    color: white;
    background: 0;
    border: 0;

    &:focus {
      border: 0;
    }
  }
}
</style>

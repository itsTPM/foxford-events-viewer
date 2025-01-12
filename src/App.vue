<script setup>
import { ref, watch } from "vue";
import Button from "./components/ui/Button.vue";

const showMode = ref("all"); // all, removed, not-removed
const isMessagesLoading = ref(false);
const messages = ref([]);
const reactions = ref([]);
const hosts = ref([]);
const room = ref(null);
const filteredMessages = ref([]);

function handleReactions(event) {
  const targetMessage = messages.value.find(
    (message) => message.label === event.data.messageId
  );

  if (targetMessage) {
    if (!targetMessage.reactions) {
      targetMessage.reactions = {};
    }

    if (!targetMessage.reactions[event.data.reactionName]) {
      targetMessage.reactions[event.data.reactionName] = 0;
    }

    if (event.removed) {
      targetMessage.reactions[event.data.reactionName]--;
    } else {
      targetMessage.reactions[event.data.reactionName]++;
    }
  }
}

function handleHosts(events) {
  const hostsSet = new Set();

  for (const event of events) {
    hostsSet.add(event.data.id);
  }

  hosts.value = Array.from(hostsSet);
}

function handleFileUpload(event) {
  isMessagesLoading.value = true;
  const file = event.target.files[0];
  const reader = new FileReader();
  reader.onload = (event) => {
    const data = JSON.parse(event.target.result);
    messages.value = data.events.filter((event) => event.type === "message");
    reactions.value = data.events.filter((event) => event.type === "reaction");
    const hostsEvents = data.events.filter((event) => event.type === "leader");

    for (const reaction of reactions.value) {
      handleReactions(reaction);
    }

    handleHosts(hostsEvents);

    room.value = data.room;
    isMessagesLoading.value = false;
  };
  reader.readAsText(file);
}

function getEmojiForReaction(reaction) {
  const map = {
    heart: "❤️",
    thumbsup: "👍",
    wave: "👋",
    grinning_face_with_big_eyes: "😃",
    slightly_frowning_face: "🙁",
    fox_face: "🦊",
    heavy_plus_sign: "➕",
    heavy_minus_sign: "➖",
    one: "1️⃣",
    two: "2️⃣",
    three: "3️⃣",
    four: "4️⃣",
    five: "5️⃣",
    six: "6️⃣",
    seven: "7️⃣",
    eight: "8️⃣",
    nine: "9️⃣",
    zero: "0️⃣",
  };

  return map[reaction] || "?";
}

watch(messages, (newMessages) => {
  filteredMessages.value = newMessages;
});

watch(showMode, (newMode) => {
  if (newMode === "all") {
    filteredMessages.value = messages.value;
  } else if (newMode === "removed") {
    filteredMessages.value = messages.value.filter(
      (message) => message.removed
    );
  } else if (newMode === "not-removed") {
    filteredMessages.value = messages.value.filter(
      (message) => !message.removed
    );
  }
});
</script>

<template>
  <header class="p-4 flex flex-col gap-4">
    <h1 class="text-4xl font-semibold">Foxford Events Viewer</h1>

    <Button as="label" class="relative focus-within:ring-4">
      <input
        type="file"
        accept=".json"
        class="opacity-0 absolute top-0 left-0 w-full h-full cursor-pointer"
        @change="handleFileUpload"
      />
      <span> Загрузить JSON файл </span>
    </Button>
  </header>
  <main class="p-4 grid grid-cols-2 gap-6 border-t">
    <section class="flex flex-col gap-6" v-if="messages.length">
      <h2 class="text-2xl font-medium">Сообщения в чате</h2>

      <div class="flex gap-4 p-4 bg-gray-200 w-fit border border-gray-300">
        <Button @click="showMode = 'all'" :active="showMode == 'all'">
          Все
        </Button>
        <Button @click="showMode = 'removed'" :active="showMode == 'removed'">
          Удалённые
        </Button>
        <Button
          @click="showMode = 'not-removed'"
          :active="showMode == 'not-removed'"
        >
          Не удалённые
        </Button>
      </div>

      <ul class="flex flex-col gap-2 bg-gray-200 border border-gray-300 p-4">
        <li
          class="p-4 border border-gray-400 bg-gray-300 flex flex-col gap-1"
          :class="{
            'bg-red-400 border-red-600 ': message.removed,
          }"
          v-for="message in filteredMessages"
        >
          <p class="font-bold text-sm">{{ message.created_by }}</p>
          <p class="break-words">
            {{ message.data.message }}
          </p>
          <p class="text-sm">
            {{
              new Date(message.created_at).toLocaleDateString("ru-RU", {
                hour: "2-digit",
                minute: "2-digit",
              })
            }}
          </p>
          <ul v-if="message.reactions" class="flex gap-2">
            <li
              v-for="(count, reaction) in message.reactions"
              :key="reaction"
              class="py-1 px-2 border border-black/10 bg-black/10 flex gap-2 text-[0.9rem] font-medium"
            >
              <span>{{ getEmojiForReaction(reaction) }}</span>
              <span>{{ count }}</span>
            </li>
          </ul>
          <p v-if="message.removed_reason" class="text-red-500">
            Reason: {{ message.removed_reason }}
          </p>
        </li>
      </ul>
    </section>

    <section class="flex gap-6 items-center" v-else-if="isMessagesLoading">
      <img src="/ios-spinner.svg" alt="" class="w-8 aspect-square" />
      <p class="text-xl">Обработка...</p>
    </section>

    <section class="flex flex-col gap-6" v-if="room">
      <h2 class="text-2xl font-medium">Информация о комнате</h2>
      <ul class="border bg-gray-200 border-gray-300 p-4">
        <li>
          <p class="font-bold">ID:</p>
          <p>{{ room.id }}</p>
        </li>
        <li>
          <p class="font-bold">Дата начала вебинара:</p>
          <p>
            {{
              new Date(room?.opened_at).toLocaleDateString("ru-RU", {
                hour: "2-digit",
                minute: "2-digit",
              })
            }}
          </p>
        </li>
      </ul>
      <hr />
      <template v-if="hosts">
        <h2 class="text-2xl font-medium">Хосты</h2>
        <ul class="border bg-gray-200 border-gray-300 p-4">
          <li v-for="host in hosts" :key="host">
            <p>{{ host }}</p>
          </li>
        </ul>
      </template>
    </section>
  </main>
</template>

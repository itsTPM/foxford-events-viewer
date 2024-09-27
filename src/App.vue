<script setup>
import { ref, watch } from "vue";
import Button from "./components/ui/Button.vue";

const showMode = ref("all"); // all, removed, not-removed
const isMessagesLoading = ref(false);
const messages = ref([]);
const filteredMessages = ref([]);

function handleFileUpload(event) {
  isMessagesLoading.value = true;
  const file = event.target.files[0];
  const reader = new FileReader();
  reader.onload = (event) => {
    const data = JSON.parse(event.target.result);
    messages.value = data.events.filter((event) => event.type === "message");
    isMessagesLoading.value = false;
  };
  reader.readAsText(file);
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
  <main class="p-4 flex flex-col gap-6">
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

    <section class="flex flex-col gap-6 border-t pt-4" v-if="messages.length">
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

      <ul
        class="flex flex-col gap-2 bg-gray-200 border border-gray-300 p-4 max-w-xl"
      >
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
          <p v-if="message.removed_reason" class="text-red-500">
            Reason: {{ message.removed_reason }}
          </p>
        </li>
      </ul>
    </section>

    <section class="flex gap-6 itesm-center" v-else-if="isMessagesLoading">
      <img src="/ios-spinner.svg" alt="" class="w-8 aspect-square" />
      <p class="text-xl">Загрузка сообщений...</p>
    </section>
  </main>
</template>

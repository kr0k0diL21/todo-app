<script setup lang="ts">
import { ref, watch, onMounted } from 'vue'

interface Todo {
  id: number
  text: string
  completed: boolean
}

const todos = ref<Todo[]>([])
const newTodo = ref('')
let nextId = 1

// localStorage
onMounted(() => {
  const saved = localStorage.getItem('todos')
  if (saved) {
    const parsed = JSON.parse(saved)
    todos.value = parsed
    nextId = Math.max(...parsed.map((t: Todo) => t.id), 0) + 1
  }
})

watch(todos, (newVal) => {
  localStorage.setItem('todos', JSON.stringify(newVal))
}, { deep: true })

// Todo hinzufügen
const addTodo = () => {
  const text = newTodo.value.trim()
  if (!text) return
  todos.value.push({ id: nextId++, text, completed: false })
  newTodo.value = ''
}

// Umschalten
const toggleTodo = (id: number) => {
  const todo = todos.value.find(t => t.id === id)
  if (todo) todo.completed = !todo.completed
}

// Löschen
const deleteTodo = (id: number) => {
  todos.value = todos.value.filter(t => t.id !== id)
}
</script>

<template>
  <div class="min-h-screen bg-gradient-to-br from-purple-50 via-pink-50 to-blue-50 p-4 md:p-8">
    <!-- Hintergrund-Dekoration – sanfter -->
    <div class="fixed inset-0 -z-10 overflow-hidden pointer-events-none">
      <div class="absolute -top-40 -right-40 w-80 h-80 bg-purple-300 rounded-full mix-blend-multiply filter blur-3xl opacity-60 animate-blob"></div>
      <div class="absolute -bottom-40 -left-40 w-80 h-80 bg-yellow-300 rounded-full mix-blend-multiply filter blur-3xl opacity-60 animate-blob animation-delay-2000"></div>
      <div class="absolute top-40 left-40 w-80 h-80 bg-pink-300 rounded-full mix-blend-multiply filter blur-3xl opacity-60 animate-blob animation-delay-4000"></div>
    </div>

    <div class="max-w-2xl mx-auto">
      <!-- Glasmorphism Card -->
      <div class="backdrop-blur-xl bg-white/70 rounded-3xl shadow-2xl p-8 border border-white/20">
        
        <!-- Header – ohne Puls, nur Gradient -->
        <h1 class="text-4xl md:text-5xl font-bold text-center bg-gradient-to-r from-purple-600 to-pink-600 bg-clip-text text-transparent mb-8">
          Deine Todos
        </h1>

        <!-- Eingabe -->
        <div class="flex gap-3 mb-8 group">
          <input
            v-model="newTodo"
            @keyup.enter="addTodo"
            type="text"
            placeholder="Was kommt als Nächstes?"
            class="flex-1 px-6 py-4 text-lg bg-white/50 backdrop-blur-md border border-purple-200 rounded-2xl focus:outline-none focus:ring-4 focus:ring-purple-300 focus:border-purple-400 transition-all duration-300 placeholder:text-gray-400"
          />
          <button
            @click="addTodo"
            class="px-8 py-4 bg-gradient-to-r from-purple-600 to-pink-600 text-white font-semibold rounded-2xl hover:from-purple-700 hover:to-pink-700 transform hover:scale-105 transition-all duration-300 shadow-lg"
          >
            Add
          </button>
        </div>

        <!-- Todo-Liste -->
        <TransitionGroup name="list" tag="ul" class="space-y-4">
          <li
            v-for="todo in todos"
            :key="todo.id"
            class="flex items-center justify-between p-5 bg-white/60 backdrop-blur-md rounded-2xl border border-purple-100 hover:bg-white/80 hover:shadow-lg transition-all duration-300 group"
          >
            <div class="flex items-center gap-4">
              <!-- Custom Checkbox -->
              <label class="relative cursor-pointer">
                <input
                  type="checkbox"
                  :checked="todo.completed"
                  @change="toggleTodo(todo.id)"
                  class="sr-only"
                />
                <div
                  :class="[
                    'w-7 h-7 rounded-full border-2 flex items-center justify-center transition-all duration-300',
                    todo.completed
                      ? 'bg-gradient-to-r from-purple-500 to-pink-500 border-transparent'
                      : 'border-purple-300 group-hover:border-purple-500'
                  ]"
                >
                  <svg
                    v-if="todo.completed"
                    class="w-4 h-4 text-white"
                    fill="none"
                    stroke="currentColor"
                    viewBox="0 0 24 24"
                  >
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7" />
                  </svg>
                </div>
              </label>

              <!-- Text -->
              <span
                :class="[
                  'text-lg font-medium transition-all duration-300',
                  todo.completed ? 'line-through text-gray-400' : 'text-gray-800'
                ]"
              >
                {{ todo.text }}
              </span>
            </div>

            <!-- Löschen – weicher Übergang -->
            <button
              @click="deleteTodo(todo.id)"
              class="opacity-50 group-hover:opacity-100 transition-opacity duration-300 text-red-400 hover:text-red-600"
            >
              <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
              </svg>
            </button>
          </li>
        </TransitionGroup>

        <!-- Leere Nachricht -->
        <p v-if="!todos.length" class="text-center text-gray-400 py-12 text-lg italic">
          Alles erledigt? Oder noch nichts los?
        </p>

        <!-- Alle löschen -->
        <div v-if="todos.length" class="mt-10 text-center">
          <button
            @click="todos = []; nextId = 1"
            class="text-sm text-purple-600 hover:text-purple-800 underline decoration-wavy"
          >
            Alle löschen
          </button>
        </div>
      </div>

      <!-- Footer -->
      <p class="text-center mt-8 text-xs text-gray-500">
        Mit Liebe & Tailwind gebaut
      </p>
    </div>
  </div>
</template>

<style scoped>
/* Sanfte Blob-Animation */
@keyframes blob {
  0%, 100% { transform: translate(0px, 0px) scale(1); }
  50% { transform: translate(20px, -30px) scale(1.05); }
}
.animate-blob {
  animation: blob 12s infinite ease-in-out;
}
.animation-delay-2000 {
  animation-delay: 2s;
}
.animation-delay-4000 {
  animation-delay: 4s;
}

/* Sanfte Liste */
.list-enter-active, .list-leave-active {
  transition: all 0.5s ease;
}
.list-enter-from, .list-leave-to {
  opacity: 0;
  transform: translateY(15px);
}
.list-move {
  transition: transform 0.5s ease;
}
</style>
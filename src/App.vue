<script>
import CheckSquare from './icons/check-square.vue';
import Square from './icons/square.vue';
import Pencil from './icons/pencil.vue';
import Trash from './icons/trash.vue';
import PlusCircle from './icons/plus-circle.vue';
import SwitchVertical from './icons/switch-vertical.vue';
import Circle from './icons/circle.vue';
import RecordCircle from './icons/record-circle.vue';
import ChevronRight from './icons/chevron-right.vue';
import ChevronBottom from './icons/chevron-bottom.vue';
import Banner from './components/banner.vue';
import axios from "axios";

export default {
  data() {
    return {
      data: {
        version: 2,
        todos: [ ],
        preferences: {
          theme: 'dark',
          sortType: 'date',
          showTodos: 'all',
          sortDesc: false,
          hideBanner: false,
          hideSearchBar: false,
          date: 'full' ,
        },
        user: {
          firstName: undefined,
        },
      },
      cache: {
        filteredTodos: undefined,
        expandPreferences: false,
        currentContent: '',
        searchContent: '',
      },
    };
  },
  watch: {
    'data.todos': {
      handler(todos) {
        this.updateTodosOnServer(todos);
      },
      deep: true,
    },
  },
  computed: {
   
     personalizedTitle() {
      return this.data.user.firstName
        ? `Welcome, ${this.data.user.firstName}`
        : 'Welcome,';
    },
    
     
    filterTodos() {
      if (this.cache.searchContent !== '') {
        return this.cache.filteredTodos.filter(todo =>
          todo.content.toLowerCase().includes(this.cache.searchContent.toLowerCase())
        );
      }

      return this.cache.filteredTodos;
    },
  },
  async created() {
    try {
      const response = await axios.get('http://localhost:3001/todos');
      this.data.todos = response.data;
    } catch (error) {
      console.error('Error fetching todos:', error);
    }
  },
  
  mounted() {
    switch (this.data.preferences.theme) {
      case 'os':
        window.matchMedia('(prefers-color-scheme: dark)').matches
          ? document.querySelector('html').classList.add('dark')
          : document.querySelector('html').classList.remove('dark');
        break;
      case 'light':
        document.querySelector('html').classList.remove('dark');
        break;
      case 'dark':
        document.querySelector('html').classList.add('dark');
        break;
    }

    this.rearrangeTodos();
  },
  
  methods: {
    
       async toggleStatus(id) {
        const todo = this.data.todos.find(todo => todo.id === id);
        todo.completed=!todo.completed;
        const response = await axios.put(`http://localhost:3001/todos/${todo.id}`, todo);
      
  },

    
    async addTodo() {
      this.cache.currentContent = this.cache.currentContent.trim();
    if (this.cache.currentContent === '') this.cache.currentContent = 'Blank to-do';

    let newTodo = {
      content: this.cache.currentContent,
      date: Date.now(),
      completed: false,
    };

    try {
      const response = await axios.post('http://localhost:3001/todos', newTodo);
      this.unshiftOrPushHandler(response.data);
      this.rearrangeTodos();
      this.cache.currentContent = '';
    } catch (error) {
      console.error('Error adding todo:', error);
    }
    },
    
     async removeTodo(id) {
      try {
        const response = await axios.delete(`http://localhost:3001/todos/${id}`);
        if (response.status === 200) {
          this.data.todos = this.data.todos.filter(todo => todo.id !== id);
        }
      } catch (error) {
        console.error('Error deleting todo:', error);
      }


      this.rearrangeTodos();
    },
    
    unshiftOrPushHandler(todoObj) {
      switch (this.data.preferences.sortType) {
        case 'name':
          this.data.preferences.sortDesc
            ? this.data.todos.push(todoObj)
            : this.data.todos.unshift(todoObj);
          break;
        default:
          this.data.preferences.sortDesc
            ? this.data.todos.unshift(todoObj)
            : this.data.todos.push(todoObj);
          break;
      }
    },
    
    formatDate(date) {
      switch (this.data.preferences.date) {
        case 'date':
          return new Date(date).toLocaleDateString();
        case 'time':
          return new Date(date).toLocaleTimeString();
        case 'full':
          return new Date(date).toLocaleString();
        default:
          return '';
      }
    },
    async updateTodosOnServer(todos) {
      try {
        await axios.put('http://localhost:3001/todos', todos);
      } catch (error) {
        console.error('Error updating todos on server:', error);
      }
    },
    
   
    rearrangeTodos() {
      switch (this.data.preferences.sortType) {
        case 'name':
          this.data.preferences.sortDesc
            ? this.data.todos.sort((a, b) => b.content.localeCompare(a.content))
            : this.data.todos.sort((a, b) => a.content.localeCompare(b.content));
          break;
        default:
          this.data.preferences.sortDesc
            ? this.data.todos.sort((a, b) => parseFloat(b.date) - parseFloat(a.date))
            : this.data.todos.sort((a, b) => parseFloat(a.date) - parseFloat(b.date));
          break;
      }

      switch (this.data.preferences.showTodos) {
        case 'active':
          this.cache.filteredTodos = this.data.todos.filter(todo => !todo.completed);
          break;
        case 'completed':
          this.cache.filteredTodos = this.data.todos.filter(todo => todo.completed);
          break;
        default:
          this.cache.filteredTodos = this.data.todos;
          break;
      }
    },
  },
  components: {
    CheckSquare,
    Square,
    Pencil,
    Trash,
    PlusCircle,
    SwitchVertical,
    Circle,
    RecordCircle,
    ChevronRight,
    ChevronBottom,
    Banner,
  },
};
</script>

<template>
  <div class="p-6 md:p-12 gap-6 md:gap-12 flex flex-col">
    <div class="w-full flex flex-col gap-3">
      <div>
        <h1 class="text-4xl font-bold dark:text-gray-200">
          {{ personalizedTitle }}
        </h1>
      </div>
      <p v-if="data.todos.length === 0" class="dark:text-gray-300">
        You didn't add any to-do. To create your first to-do, type below then hit enter or
        the + sign.
      </p>
      
      <div class="w-full flex gap-3 md:gap-6" v-else>
        <div class="w-1/12 flex justify-center"></div>
        <div class="w-10/12 flex justify-end">
          <input
            type="text"
            name="searchTodo"
            id="searchTodo"
            class="rounded px-3 bg-gray-100 dark:bg-gray-700 dark:text-white w-full md:w-auto"
            placeholder="Search todo"
            v-if="!data.preferences.hideSearchBar"
            v-model="cache.searchContent"
          />
        </div>
        <div class="w-1/12 flex gap-3 justify-center">
          <a
            href="#"
            @click.prevent
            @click="
              data.preferences.sortDesc = !data.preferences.sortDesc;
              rearrangeTodos();
            "
            class="text-gray-500 dark:text-gray-400 hover:text-gray-700 dark:hover:text-gray-200 transition-all duration-200"
          >
            <SwitchVertical />
          </a>
        </div>
      </div>
      <div class="w-full flex flex-col gap-3">
        <div
          class="w-full flex gap-3 md:gap-6"
          v-for="todo in filterTodos"
          :key="todo.id"
        >
          <div
            class="w-1/12 flex justify-center"
            :class="
              data.preferences.date !== 'hide' && data.preferences.date ? 'mt-2' : 'mt-0'
            "
          >
            <a
              href="#"
              @click.prevent
              @click="toggleStatus(todo.id)"
              class="text-gray-500 hover:text-gray-700 dark:hover:text-gray-300 transition-all duration-200"
            >
              <CheckSquare v-if="todo.completed" />
              <Square v-else />
            </a>
          </div>
          <div class="w-10/12">
            <input
              type="text"
              :name="todo.id"
              :id="todo.id"
              v-model="todo.content"
              class="w-full dark:bg-gray-800 dark:text-gray-300"
              :class="{ 'line-through text-gray-400 dark:text-gray-600': todo.completed }"
            />
            <p
              v-if="data.preferences.date !== 'hide' && data.preferences.date"
              class="text-xs text-gray-800 dark:text-gray-400"
              :class="{ 'text-gray-300 dark:text-gray-700': todo.completed }"
            >
              {{ formatDate(todo.date) }}
            </p>
          </div>
          <div
            class="w-1/12 flex gap-3 justify-center"
            :class="
              data.preferences.date !== 'hide' && data.preferences.date ? 'mt-2' : 'mt-0'
            "
          >
            <a
              href="#"
              @click.prevent
              @click="removeTodo(todo.id)"
              class="text-gray-300 dark:text-gray-600 hover:text-red-500 dark:hover:text-red-400 transition-all duration-200"
            >
              <Trash />
            </a>
          </div>
        </div>
      </div>

      <div class="w-full flex gap-3 md:gap-6">
        <div class="w-1/12 flex justify-center"></div>
        <div class="w-10/12">
          <input
            type="text"
            name="todoContent"
            id="todoContent"
            class="rounded w-full bg-gray-100 dark:bg-gray-700 dark:text-white px-3"
            v-model="cache.currentContent"
            v-on:keyup.enter="addTodo"
            placeholder="Add new todo from here..."
          />
        </div>
        <div class="w-1/12 flex gap-3 justify-center">
          <a
            href="#"
            @click.prevent
            @click="addTodo"
            class="text-gray-700 dark:text-gray-200 hover:text-green-700 dark:hover:text-green-200 transition-all duration-200"
          >
            <PlusCircle />
          </a>
        </div>
      </div>
    </div>
    <div class="flex flex-col gap-6">
      <a
        href="#"
        @click.prevent
        @click="cache.expandPreferences = !cache.expandPreferences"
      >
        <h2
          class="text-md text-gray-700 hover:text-black dark:text-gray-400 dark:hover:text-gray-200 transition-all duration-200 flex gap-3"
        >
          <ChevronBottom v-if="cache.expandPreferences" />
          <ChevronRight v-else /> Preferences
        </h2>
      </a>
      <div class="flex flex-col gap-6" v-if="cache.expandPreferences">
        <div class="flex gap-6">
          <p class="dark:text-gray-400 text-sm">First name:</p>
          <input
            type="text"
            name="firstName"
            id="firstName"
            class="rounded bg-gray-100 dark:bg-gray-700 dark:text-white px-3"
            placeholder="Your first name"
            v-model.trim="data.user.firstName"
          />
        </div>
        <a
          href="#"
          @click.prevent
          class="text-gray-700 hover:text-black dark:text-gray-400 dark:hover:text-gray-200 transition-all duration-200 flex gap-3 text-sm"
          @click="data.preferences.hideSearchBar = !this.data.preferences.hideSearchBar"
        >
          <CheckSquare v-if="data.preferences.hideSearchBar" height="20" width="20" />
          <Square v-else height="20" width="20" />
          <p>Hide search bar</p>
        </a>
        <p class="dark:text-gray-400 text-sm">Date/Time:</p>
        <div class="flex gap-6">
          <a
            href="#"
            @click.prevent
            class="text-gray-700 hover:text-black dark:text-gray-400 dark:hover:text-gray-200 transition-all duration-200 flex gap-3 text-sm"
            @click="data.preferences.date = 'hide'"
          >
            <RecordCircle
              v-if="
                data.preferences.date === 'hide' || data.preferences.date === undefined
              "
            />
            <Circle v-else />
            <p>Hide</p>
          </a>
          <a
            href="#"
            @click.prevent
            class="text-gray-700 hover:text-black dark:text-gray-400 dark:hover:text-gray-200 transition-all duration-200 flex gap-3 text-sm"
            @click="data.preferences.date = 'date'"
          >
            <RecordCircle v-if="data.preferences.date === 'date'" />
            <Circle v-else />
            <p>Date</p>
          </a>
          <a
            href="#"
            @click.prevent
            class="text-gray-700 hover:text-black dark:text-gray-400 dark:hover:text-gray-200 transition-all duration-200 flex gap-3 text-sm"
            @click="data.preferences.date = 'time'"
          >
            <RecordCircle v-if="data.preferences.date === 'time'" />
            <Circle v-else />
            <p>Time</p>
          </a>
          <a
            href="#"
            @click.prevent
            class="text-gray-700 hover:text-black dark:text-gray-400 dark:hover:text-gray-200 transition-all duration-200 flex gap-3 text-sm"
            @click="data.preferences.date = 'full'"
          >
            <RecordCircle v-if="data.preferences.date === 'full'" />
            <Circle v-else />
            <p>Full</p>
          </a>
        </div>
        <p class="dark:text-gray-400 text-sm">Sort type:</p>
        <div class="flex gap-6">
          <a
            href="#"
            @click.prevent
            class="text-gray-700 hover:text-black dark:text-gray-400 dark:hover:text-gray-200 transition-all duration-200 flex gap-3 text-sm"
            @click="
              data.preferences.sortType = 'date';
              rearrangeTodos();
            "
          >
            <RecordCircle
              v-if="
                data.preferences.sortType === 'date' ||
                data.preferences.sortType === undefined
              "
            />
            <Circle v-else />
            <p>Date</p>
          </a>
          <a
            href="#"
            @click.prevent
            class="text-gray-700 hover:text-black dark:text-gray-400 dark:hover:text-gray-200 transition-all duration-200 flex gap-3 text-sm"
            @click="
              data.preferences.sortType = 'name';
              rearrangeTodos();
            "
          >
            <RecordCircle v-if="data.preferences.sortType === 'name'" />
            <Circle v-else />
            <p>Name</p>
          </a>
        </div>
        <p class="dark:text-gray-400 text-sm">Show todos:</p>
        <div class="flex gap-6">
          <a
            href="#"
            @click.prevent
            class="text-gray-700 hover:text-black dark:text-gray-400 dark:hover:text-gray-200 transition-all duration-200 flex gap-3 text-sm"
            @click="
              data.preferences.showTodos = 'all';
              rearrangeTodos();
            "
          >
            <RecordCircle
              v-if="
                data.preferences.showTodos === 'all' ||
                data.preferences.showTodos === undefined
              "
            />
            <Circle v-else />
            <p>All</p>
          </a>
          <a
            href="#"
            @click.prevent
            class="text-gray-700 hover:text-black dark:text-gray-400 dark:hover:text-gray-200 transition-all duration-200 flex gap-3 text-sm"
            @click="
              data.preferences.showTodos = 'active';
              rearrangeTodos();
            "
          >
            <RecordCircle v-if="data.preferences.showTodos === 'active'" />
            <Circle v-else />
            <p>Active</p>
          </a>
          <a
            href="#"
            @click.prevent
            class="text-gray-700 hover:text-black dark:text-gray-400 dark:hover:text-gray-200 transition-all duration-200 flex gap-3 text-sm"
            @click="
              data.preferences.showTodos = 'completed';
              rearrangeTodos();
            "
          >
            <RecordCircle v-if="data.preferences.showTodos === 'completed'" />
            <Circle v-else />
            <p>Completed</p>
          </a>
        </div>
      </div>
    </div>
  </div>
  <div class="mt-4"></div>
  <Banner v-if="!data.preferences.hideBanner" />
</template>

<style scoped></style>

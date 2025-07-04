<template>
  <div id="app">
    <h1>TODO List</h1>
    <div>
      <input v-model="newTodo" placeholder="Add a new TODO" @keyup.enter="addTodo" />
      <button @click="addTodo">Add</button>
    </div>
    <ul>
      <li v-for="todo in todos" :key="todo.id">
        {{ todo.text }}
        <button @click="editTodo(todo)">Edit</button>
        <button @click="deleteTodo(todo.id)">Delete</button>
      </li>
    </ul>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'App',
  data() {
    return {
      newTodo: '',
      todos: []
    };
  },
  methods: {
    async fetchTodos() {
      try {
        const response = await axios.get('/api/todo');
        this.todos = response.data;
      } catch (error) {
        console.error('Error fetching todos:', error);
      }
    },
    async addTodo() {
      if (this.newTodo.trim()) {
        try {
          const response = await axios.post('/api/todo', { text: this.newTodo, isCompleted: false });
          this.todos.push(response.data);
          this.newTodo = '';
        } catch (error) {
          console.error('Error adding todo:', error);
        }
      }
    },
    async editTodo(todo) {
      const newText = prompt('Edit TODO:', todo.text);
      if (newText !== null && newText.trim()) {
        try {
          await axios.put(`/api/todo/${todo.id}`, { ...todo, text: newText });
          todo.text = newText;
        } catch (error) {
          console.error('Error updating todo:', error);
        }
      }
    },
    async deleteTodo(id) {
      try {
        await axios.delete(`/api/todo/${id}`);
        this.todos = this.todos.filter(todo => todo.id !== id);
      } catch (error) {
        console.error('Error deleting todo:', error);
      }
    }
  },
  mounted() {
    this.fetchTodos();
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
input {
  padding: 5px;
  margin-right: 10px;
}
button {
  padding: 5px 10px;
  margin: 0 5px;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  margin: 10px 0;
}
</style>

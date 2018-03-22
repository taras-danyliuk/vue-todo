<template>
  <section class="todoapp">
    <header class="header">
      <h1>Todos</h1>
      <input class="new-todo"
             autofocus autocomplete="off"
             placeholder="Type new todo here"
             v-model="newTodo"
             @keyup.enter="addTodo">
    </header>

    <section class="main" v-show="todos.length" v-cloak>
      <ul class="todo-list">
        <li v-for="todo in filteredTodos"
            class="todo"
            :key="todo.id"
            :class="{ completed: todo.completed, editing: todo === editedTodo }">
          <div class="view">
            <input class="toggle" type="checkbox" v-model="todo.completed"/>

            <label @dblclick="editTodo(todo)">{{ todo.title }}</label>

            <button class="destroy" @click="removeTodo(todo)"></button>
          </div>

          <input
            class="edit" type="text"
            v-model="todo.title"
            v-todo-focus="todo == editedTodo"
            @blur="doneEdit(todo)"
            @keyup.enter="doneEdit(todo)"
            @keyup.esc="cancelEdit(todo)"
          />
        </li>
      </ul>
    </section>

    <footer class="footer" v-show="todos.length" v-cloak>
      <ul class="filters">
        <li><a @click="changeFilter('all')" :class="{ selected: visibility === 'all' }">All</a></li>
        <li><a @click="changeFilter('active')" :class="{ selected: visibility === 'active' }">Active</a></li>
        <li><a @click="changeFilter('completed')" :class="{ selected: visibility === 'completed' }">Completed</a></li>
      </ul>
    </footer>
  </section>
</template>

<script>

  // localStorage persistence
  const STORAGE_KEY = 'todos-vuejs-2.0';
  const todoStorage = {
    fetch: function () {
      const todos = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]');

      todos.forEach(function (todo, index) {
        todo.id = index
      });
      todoStorage.uid = todos.length;

      return todos;
    },
    save: function (todos) {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(todos))
    }
  };

  // visibility filters
  const filters = {
    all: function (todos) {
      return todos
    },
    active: function (todos) {
      return todos.filter(function (todo) {
        return !todo.completed
      })
    },
    completed: function (todos) {
      return todos.filter(function (todo) {
        return todo.completed
      })
    }
  };


  export default {
    name: 'App',
    data: function() {
      return {
        todos: todoStorage.fetch(),
        newTodo: '',
        editedTodo: null,
        visibility: 'all'
      }
    },
    // watch todos change for localStorage persistence
    watch: {
      todos: {
        handler: function (todos) {
          todoStorage.save(todos)
        },
        deep: true
      }
    },
    // computed properties
    computed: {
      filteredTodos: function () {
        return filters[this.visibility](this.todos)
      },
    },
    // methods that implement data logic.
    // note there's no DOM manipulation here at all.
    methods: {
      addTodo: function () {
        const value = this.newTodo && this.newTodo.trim();

        if (!value) {
          return
        }

        this.todos.push({
          id: todoStorage.uid++,
          title: value,
          completed: false
        });
        this.newTodo = ''
      },

      removeTodo: function (todo) {
        this.todos.splice(this.todos.indexOf(todo), 1)
      },

      editTodo: function (todo) {
        this.beforeEditCache = todo.title;
        this.editedTodo = todo
      },

      doneEdit: function (todo) {
        if (!this.editedTodo) {
          return;
        }

        this.editedTodo = null;
        todo.title = todo.title.trim();
        if (!todo.title) {
          this.removeTodo(todo);
        }
      },

      cancelEdit: function (todo) {
        this.editedTodo = null;
        todo.title = this.beforeEditCache;
      },

      changeFilter: function (filter) {
        this.visibility = filter;
      }
    },
    directives: {
      'todo-focus': function (el, binding) {
        if (binding.value) {
          el.focus()
        }
      }
    }
  }
</script>

<style>
  html,
  body {
    margin: 0;
    padding: 0;
  }

  button {
    margin: 0;
    padding: 0;
    border: 0;
    background: none;
    font-size: 100%;
    vertical-align: baseline;
  }

  body {
    font-size: 14px;
    line-height: 1.4em;
    background: #b9b9b9;
    color: #4d4d4d;
    min-width: 230px;
    max-width: 550px;
    margin: 0 auto;
    font-weight: 300;
  }

  :focus {
    outline: 0;
  }

  .todoapp {
    background: #fff;
    margin: 130px 0 40px 0;
    position: relative;
  }

  .todoapp h1 {
    position: absolute;
    top: -110px;
    width: 100%;
    font-size: 100px;
    font-weight: 100;
    text-align: center;
    margin: 0;
    line-height: 100px;
  }

  .new-todo,
  .edit {
    position: relative;
    margin: 0;
    width: 100%;
    font-size: 24px;
    line-height: 1.4em;
    color: inherit;
    padding: 6px;
    border: 1px solid #999;
    box-sizing: border-box;
  }

  .new-todo {
    padding: 16px 16px 16px 60px;
    border: none;
    background: rgba(0, 0, 0, 0.003);
  }

  .main {
    position: relative;
    z-index: 2;
    border-top: 1px solid #e6e6e6;
  }

  .todo-list {
    margin: 0;
    padding: 0;
    list-style: none;
  }

  .todo-list li {
    position: relative;
    font-size: 24px;
    border-bottom: 1px solid #ededed;
  }

  .todo-list li:last-child {
    border-bottom: none;
  }

  .todo-list li.editing {
    border-bottom: none;
    padding: 0;
  }

  .todo-list li.editing .edit {
    display: block;
    width: 506px;
    padding: 12px 16px;
    margin: 0 0 0 43px;
  }

  .todo-list li.editing .view {
    display: none;
  }

  .todo-list li .toggle {
    text-align: center;
    width: 40px;
    /* auto, since non-WebKit browsers doesn't support input styling */
    height: auto;
    position: absolute;
    top: 0;
    bottom: 0;
    margin: auto 0;
    border: none; /* Mobile Safari */
    -webkit-appearance: none;
    appearance: none;
  }

  .todo-list li .toggle:after {
    content: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="40" height="40" viewBox="-10 -18 100 135"><circle cx="50" cy="50" r="50" fill="none" stroke="#ededed" stroke-width="3"/></svg>');
  }

  .todo-list li .toggle:checked:after {
    content: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="40" height="40" viewBox="-10 -18 100 135"><circle cx="50" cy="50" r="50" fill="none" stroke="#bddad5" stroke-width="3"/><path fill="#5dc2af" d="M72 25L42 71 27 56l-4 4 20 20 34-52z"/></svg>');
  }

  .todo-list li label {
    word-break: break-all;
    padding: 15px 60px 15px 15px;
    margin-left: 45px;
    display: block;
    line-height: 1.2;
    transition: color 0.4s;
  }

  .todo-list li.completed label {
    color: #d9d9d9;
    text-decoration: line-through;
  }

  .todo-list li .destroy {
    display: block;
    position: absolute;
    top: 0;
    right: 10px;
    bottom: 0;
    width: 40px;
    height: 40px;
    margin: auto 0 11px;
    font-size: 30px;
    color: black;
    transition: color 0.2s ease-out;
  }

  .todo-list li .destroy:after {
    content: '×';
  }

  .todo-list li .edit {
    display: none;
  }

  .todo-list li.editing:last-child {
    margin-bottom: -1px;
  }

  .footer {
    color: #777;
    padding: 10px 15px;
    height: 20px;
    text-align: center;
    border-top: 1px solid #e6e6e6;
  }

  .todo-count strong {
    font-weight: 300;
  }

  .filters {
    margin: 0;
    padding: 0;
    list-style: none;
    position: absolute;
    right: 0;
    left: 0;
  }

  .filters li {
    display: inline;
  }

  .filters li a {
    color: inherit;
    margin: 3px;
    padding: 3px 7px;
    text-decoration: none;
    border: 1px solid transparent;
    border-radius: 3px;
  }

  .filters li a:hover {
    border-color: rgba(175, 47, 47, 0.1);
  }

  .filters li a.selected {
    border-color: rgba(175, 47, 47, 0.2);
  }

  .info p {
    line-height: 1;
  }

  .info a {
    color: inherit;
    text-decoration: none;
    font-weight: 400;
  }

  .info a:hover {
    text-decoration: underline;
  }

  /*
      Hack to remove background from Mobile Safari.
      Can't use it globally since it destroys checkboxes in Firefox
  */
  @media screen and (-webkit-min-device-pixel-ratio:0) {
    .todo-list li .toggle {
      background: none;
    }

    .todo-list li .toggle {
      height: 40px;
    }
  }

</style>

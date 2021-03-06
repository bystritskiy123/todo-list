<template>
  <div>
    <section class="section">
      <div class="container">
        <div class="level">
          <b-button class="is-primary" @click="isAddModalActive = true">
            Add A Todo
          </b-button>
          <b-button class="is-warning" @click="deleteAllTodos">
            Delete All Todos
          </b-button>
        </div>

        <b-table :data="todos" default-sort="priority">
          <template slot-scope="todos">
            <b-table-column field="id" label="ID" width="40" sortable numeric>
              {{ todos.row.id }}
            </b-table-column>

            <b-table-column field="todo" label="Todo" sortable>
              {{ todos.row.todo }}
            </b-table-column>

            <b-table-column field="priority" label="Priority" sortable>
              {{ todos.row.priority }}
            </b-table-column>

            <b-table-column label="Edit">
              <b-button
                type="is-text"
                icon-left="settings-outline"
                @click="openEditModal(todos.row)"
              >settings</b-button>
            </b-table-column>

            <b-table-column label="Delete">
              <b-button
                type="is-text"
                icon-left="delete"
                @click="deleteTodo(todos.row)"
              ></b-button>
            </b-table-column>
          </template>
        </b-table>
      </div>
    </section>

    <b-modal :active.sync="isEditModalActive" has-modal-card>
      <todo-edit-modal
        :todo="selectedTodo"
        :priorities="priorities"
        @edit-todo="onEditTodo"
      ></todo-edit-modal>
    </b-modal>

    <b-modal :active.sync="isAddModalActive" has-modal-card>
      <todo-add-modal
        @add-todo="onAddTodo"
        :priorities="priorities"
      ></todo-add-modal>
    </b-modal>
  </div>
</template>

<script>
import TodoEditModal from "@/components/TodoEditModal";
import TodoAddModal from "@/components/TodoAddModal";

export default {
  name: "TodoTable",
  components: { TodoEditModal, TodoAddModal },
  data() {
    return {
      initialTodos: [
        {
          id: 1,
          todo: "Consider Adam as a candidate",
          priority: "life changing"
        },
        {
          id: 2,
          todo: "Take out the garbage",
          priority: "life changing"
        },
        {
          id: 3,
          todo: "Paint the house",
          priority: "important"
        },
        {
          id: 4,
          todo: "Replace the doorknob",
          priority: "meh"
        },
        {
          id: 5,
          todo: "Smell the roses",
          priority: "important"
        }
      ],
      todos: [],
      priorities: [
        { id: 1, name: "life changing" },
        { id: 2, name: "important" },
        { id: 3, name: "meh" }
      ],
      isEditModalActive: false,
      selectedTodo: {},
      isAddModalActive: false
    };
  },
  mounted() {
    if (localStorage.getItem("todos")) {
      this.todos = JSON.parse(localStorage.getItem("todos"));
    } else {
      this.todos = this.initialTodos;
    }
  },
  methods: {
    openEditModal(todo) {
      this.selectedTodo = todo;
      this.isEditModalActive = true;
    },
    onAddTodo(item) {
      // get the highest number id to iterate on it
      const highestId = Math.max.apply(Math, this.todos.map(item => item.id));
      // Add the item to the array
      this.todos.push({
        id: highestId + 1,
        todo: item.title,
        priority: item.priority
      });
      // save the updated array in localstorage
      this.saveLocalStorageTodos();
      this.isAddModalActive = false;
    },
    onEditTodo(item) {
      const todo = this.findTodo(item);
      // Apply the updated values
      todo.todo = item.todo;
      todo.priority = item.priority;
      // save the updated array in localstorage
      this.saveLocalStorageTodos();
      // close the modal
      this.isEditModalActive = false;
    },
    deleteTodo(item) {
      this.$buefy.dialog.confirm({
        title: `Deleting Todo`,
        confirmText: "Delete Todo",
        type: "is-danger",
        hasIcon: true,
        message: `Are you sure you want to delete this item? This cannot be undone.`,
        onConfirm: () => {
          // find in the array and remove
          const index = this.todos.indexOf(item);
          this.todos.splice(index, 1);
          // save the updated array in localstorage
          this.saveLocalStorageTodos();
        }
      });
    },
    deleteAllTodos() {
      this.$buefy.dialog.confirm({
        title: `Deleting Todos`,
        confirmText: "Delete Todos",
        type: "is-danger",
        hasIcon: true,
        message: `Are you sure you want to delete all the todos on your list? This cannot be undone.`,
        onConfirm: () => {
          this.todos = [];
          // save the updated array in localstorage
          this.saveLocalStorageTodos();
        }
      });
    },
    findTodo(item) {
      return this.todos.find(todo => todo.id === item.id);
    },
    saveLocalStorageTodos() {
      localStorage.setItem("todos", JSON.stringify(this.todos));
      this.todos = JSON.parse(localStorage.getItem("todos"));
    }
  }
};
</script>

<style lang="scss" scoped></style>

<template>
  <div class="formInput">
    <input 
	class="input" 
	placeholder="What needs to be done?" 
	v-model="newTodo"
	@keyup.enter="addTodo"
    />
    <i class="downArrow fa fa-angle-right" />
  </div>
</template>

<script>
import { GET_MY_TODOS } from "./TodoPrivateList.vue";

import gql from "graphql-tag";

const ADD_TODO = gql`
  mutation ($todo: String!, $isPublic: Boolean!) {
  insert_todos(objects: {title: $todo, is_public: $isPublic}) {
    affected_rows
    returning {
      id
      title
      created_at
      is_completed
    }
  }
}`;

  export default {
    props: ['type'],
    data() {
      return {
        newTodo: '',
      }
    },
    methods: {
      addTodo: function () {

        // insert new todo into db
        const title = this.newTodo && this.newTodo.trim()
        const ispublic = this.type === "public" ? true : false;
        this.$apollo.mutate({
          mutation: ADD_TODO,
          variables: 
          {todo: title,
          isPublic: ispublic
          },
          update: (cache, {data: { insert_todos} })=> {
            try {
              if (this.type === "private") {
                const data = cache.readQuery({
                  query: GET_MY_TODOS
                });
                const insertedTodo = insert_todos.returning;
                data.todos.splice(0,0,insertedTodo[0]);
                cache.writeQuery({
                  query: GET_MY_TODOS,
                  data
                });
              }
            }
            catch (e) {
              console.error(e);
            }
          },
        });
      
      
      
      
      
      this.newTodo = '';
      },
    }
  }
</script>

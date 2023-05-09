<template>
<div class="rush-todo">
  <div class="title has-text-centered">
    Rush App
  </div>

  <form
    @submit.prevent="addTodo" 
  >
    <div class="field is-grouped mb-5">
    <p class="control is-expanded">
      <input 
      v-model = "newTodoContent"
      class="input" 
      type="text" 
      placeholder="Add something">
    </p>
    <p class="control">
      <button 
      :disabled="!newTodoContent"
      class="button is-info">
        Add
      </button>
    </p>
    </div>
  </form>

    <div 
    v-for="todo in todos" 
    :key="todo.id" 
    class="card mb-5"
    :class = "{'has-background-success-light' : todo.done}" 
    >
    <div class="card-content">
      <div class="content">

        <div class="columns is-mobile is-vcentered">
          <div 
          class="column"
          :class="{'has-text-success line-through' : todo.done}"
          >
            {{ todo.content }}
          </div>
          <div class="column is-5 has-text-right">
            <button 
            @click="toggleDone(todo.id)"
            class="button"
            :class="todo.done ? 'is-success' : 
            'is-light'"
            >
              &check;
            </button>
            <button 
            @click="deleteTodo(todo.id)"
            class="button is-danger ml-2">
              &cross;
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>

</div>
</template>

<script setup>
  import { ref, onMounted } from 'vue'
  import { 
    collection, onSnapshot, addDoc, deleteDoc, doc, 
    updateDoc, query, orderBy} 
  from 'firebase/firestore';
  import { db } from '@/firebase'

  // firebase ref

  const todosCollectionRef = collection(db, 'todos')
  const todosCollectionQuery = query(todosCollectionRef, orderBy('date', 'desc'))

  const todos = ref([
    // {
    //   id: 'id1',
    //   content: 'content 1',
    //   done: false
    // },
    // {
    //   id: 'id2',
    //   content: 'content 2',
    //   done: false
    // }
  ])

  // get todos

  onMounted( () => {
    // const querySnapshot = await getDocs(collection(db, 'todos'))
    // let fbTodos = []
    //   querySnapshot.forEach((doc) => {
    //   console.log(doc.id, " => ", doc.data())
    //   const todo = {
    //     id: doc.id,
    //     content: doc.data().content,
    //     done: doc.data().done
    //   }
    //   fbTodos.push(todo)
    //   })
    //   todos.value = fbTodos

      onSnapshot(todosCollectionQuery, (querySnapshot) => {
      const fbTodos = []
      querySnapshot.forEach((doc) => {
          const todo = {
            id: doc.id,
            content: doc.data().content,
            done: doc.data().done
        }
        fbTodos.push(todo)
      })
      todos.value = fbTodos
    })
})

  // add todo

  const newTodoContent = ref('')
  const addTodo = () => {
    const docRef = addDoc(todosCollectionRef, {
      content: newTodoContent.value,
      done: false,
      date: Date.now()
    })
    // const newTodo = {
    //   id: uuidv4(),
    //   content: newTodoContent.value,
    //   done: false
    // }
    // todos.value.unshift(newTodo)
    newTodoContent.value = ''
  }

   // delete todo

   const deleteTodo = id => {
    deleteDoc(doc(todosCollectionRef, id));
   }

  //  toggle done
   
  const toggleDone = id => {
  //   const index = todos.value.findIndex(todo => todo.id 
  //   === id)
  // todos.value[index].done = !todos.value[index].done

  const index = todos.value.findIndex(todo => todo.id === id)

  // Set the "capital" field of the city 'DC'
  updateDoc(doc(todosCollectionRef, id), {
    done: !todos.value[index].done
  })
}

</script>


<style>
@import 'bulma/css/bulma.min.css';

.rush-todo {
  max-width: 400px;
  padding: 20px;
  margin: 0 auto;
}

.line-through {
  text-decoration: line-through;
}
</style>

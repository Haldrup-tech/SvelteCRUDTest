<script>
import { initializeApp } from "firebase/app";
import { getFirestore, collection, query, where, onSnapshot, doc, updateDoc, deleteDoc, addDoc } from "firebase/firestore";
import { firebaseConfig } from "$lib/firebaseConfig";


const firebaseApp = initializeApp(firebaseConfig);
const db = getFirestore();

const colRef = collection(db, "Todos");
const unsubscribe = onSnapshot(colRef, (querySnapshot) => {
  let fbTodos = [];
  querySnapshot.forEach((doc) => {
    let todo = {...doc.data(), id: doc.id}
    fbTodos = [todo, ...fbTodos];
  });
  todos = fbTodos;
})



  let todos = [];
  let taskInput = "";
  let error = "";

  const addTodo = async () => {
    if (taskInput !== "") {
      await addDoc(collection(db, "Todos"), {
        task: taskInput,
        isComplete: false,
        createdAt: new Date(),
      })
      error = "";
    } else {
      error = "Task is empty";
    }
    taskInput = ""
  };

  const markTodoAsComplete = async (item) => {
    await updateDoc(doc(db, "Todos", item.id), {
      isComplete: !item.isComplete,
    });
  };

  const deleteTodo = async (item) => {
    await deleteDoc(doc(db, "Todos", item.id));
  }

  const keyIsPressed = (event) => {
    if (event.key == "Enter" ) {
      addTodo();
    }
  }

</script>

<input type="text" placeholder="Add a task" bind:value={taskInput}>
<button on:click={addTodo}>Add</button>
<p class="error">{error}</p>
<ol>
  {#each todos as item}
    <li>
      <span class:complete={item.isComplete}>
        {item.task}
      </span>
      <span>
        <button on:click={() => markTodoAsComplete(item)}>✔️</button>
      </span>
      <span>
        <button on:click={() => deleteTodo(item)}>✖️</button>
      </span>
    </li>
  {:else}
    <p>no todos found</p>
  {/each}
</ol>

<svelte:window on:keydown={keyIsPressed}/>

<style>
  .complete {
    text-decoration: line-through;
  }
  .error {
    color: red;
  }
</style>
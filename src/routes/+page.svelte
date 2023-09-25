<script lang="ts">
  import { onMount, afterUpdate } from "svelte";
  import { v4 as uuidv4 } from "uuid";

  import imgNotionNotes from "$lib/assets/oc-taking-note.png";
  import imgNotionPapers from "$lib/assets/oc-hand-holding-papers.png";

  import Trash from "../components/icons/Trash.svelte";
  import PencilSquare from "../components/icons/PencilSquare.svelte";
  import CheckSquare from "../components/icons/CheckSquare.svelte";
  import XLg from "../components/icons/XLg.svelte";
  import Floppy from "../components/icons/Floppy.svelte";

  interface ITask {
    id: string;
    name: string;
    created_on: string;
    is_done: boolean;
    is_deleted: boolean;
  }

  const defaultTask = {
    id: "",
    name: "",
    created_on: "",
    is_done: false,
    is_deleted: false,
  };

  let taskList: ITask[] = [];
  let task: string = "";
  let selected: ITask = defaultTask;

  $: unableAdd = task.length > 2 ? false : true;

  $: unableSaveUpdate = selected.name.length > 2 ? false : true;

  $: todoList = taskList
    .filter((task) => task.is_done === false && task.is_deleted === false)
    .sort(
      (a: ITask, b: ITask) => +new Date(b.created_on) - +new Date(a.created_on)
    );

  $: completedList = taskList
    .filter((task) => task.is_done === true && task.is_deleted === false)
    .sort(
      (a: ITask, b: ITask) => +new Date(b.created_on) - +new Date(a.created_on)
    );

  onMount(() => {
    const store: ITask[] | null = JSON.parse(
      (localStorage.getItem("svelte-app") as any) || null
    );

    if (store !== null) {
      taskList = [...store];
    }
  });

  // afterUpdate(() => {
  //   console.log('did update')
  // });

  const handleSubmit: () => void = () => {
    let newTask: ITask = {
      id: uuidv4(),
      name: task,
      created_on: new Date().toLocaleString(),
      is_done: false,
      is_deleted: false,
    };

    taskList.push(newTask);

    taskList = taskList;

    localStorage.setItem("svelte-app", JSON.stringify(taskList));

    task = "";
  };

  const setSelected: (id: string) => void = (id) => {
    const find = taskList.filter((item) => item.id === id)[0];

    selected = find;
  };

  const handleChangeSelectedName: (name: string) => void = (name) => {
    const editedSelected: ITask = {
      ...selected,
      name: name,
    };

    selected = editedSelected;
  };

  const resetSelected: () => void = () => {
    selected = defaultTask;
  };

  const saveSelected: () => void = () => {
    const updatedSelected = taskList.map((task) => {
      if (task.id === selected.id) {
        return selected;
      } else {
        return task;
      }
    });

    taskList = updatedSelected;

    localStorage.setItem("svelte-app", JSON.stringify(updatedSelected));

    selected = defaultTask;
  };

  const handleDone: (id: string) => void = (id) => {
    const setDone = taskList.map((task) => {
      if (task.id === id) {
        return {
          ...task,
          is_done: true,
        };
      } else {
        return task;
      }
    });

    localStorage.setItem("svelte-app", JSON.stringify(setDone));

    taskList = setDone;
  };

  const handleDelete: (id: string) => void = (id) => {
    const updateWithDeleted = taskList.map((task) => {
      if (task.id === id) {
        return {
          ...task,
          is_deleted: true,
        };
      } else {
        return task;
      }
    });

    localStorage.setItem("svelte-app", JSON.stringify(updateWithDeleted));

    taskList = updateWithDeleted;
  };
</script>

<section class="s-todo">
  <div
    class="w-full md:w-[480px] px-5 md:px-0 mx-auto py-8 md:py-16 flex flex-col gap-10"
  >
    <!-- part a -->
    <div>
      <h1 class="text-xl font-semibold mb-4">Todo</h1>
      <form class="flex gap-4" on:submit={handleSubmit}>
        <input
          type="text"
          class="py-2 px-3 block w-full border-gray-200 rounded-md text-base focus:border-blue-500 focus:ring-blue-500"
          placeholder="Add your todo list"
          bind:value={task}
        />
        <button
          type="submit"
          class="py-2 px-3 inline-block justify-center items-center whitespace-nowrap gap-2 rounded-md bg-blue-100 border border-transparent font-semibold text-blue-500 hover:text-white hover:bg-blue-500 focus:outline-none focus:ring-2 ring-offset-white focus:ring-blue-500 focus:ring-offset-2 transition-all text-sm disabled:hover:bg-blue-100 disabled:hover:text-blue-500"
          disabled={unableAdd}
        >
          Add task
        </button>
      </form>
    </div>

    <!-- part b -->
    <div class="mt-4">
      <h2 class="text-xl font-semibold">Ongoing task</h2>
      <div>
        {#if todoList.length <= 0}
          <!-- empty state -->
          <div class="w-full text-center">
            <div class="w-full py-8 px-5">
              <img src={imgNotionNotes} alt="notes" class="w-[100px] mx-auto" />
            </div>
            <p>Nothing to do right now.</p>
          </div>
        {:else}
          <div class="flex flex-col gap-4 mt-4">
            {#each todoList as { id, name }}
              <div class="flex gap-2">
                <input
                  type="text"
                  placeholder="Add your todo list"
                  class="py-2 px-3 block w-full rounded-md text-base focus:border-blue-500 focus:ring-blue-500 {selected.id ===
                  id
                    ? 'border-gray-200'
                    : 'border-transparent pointer-events-none'}"
                  bind:value={name}
                  on:input={() => handleChangeSelectedName(name)}
                />
                {#if selected.id === id}
                  <button
                    type="button"
                    class="py-2 px-3 inline-block justify-center items-center whitespace-nowrap gap-2 rounded-md bg-green-100 border border-transparent font-semibold text-green-500 hover:text-white hover:bg-green-500 focus:outline-none focus:ring-2 ring-offset-white focus:ring-green-500 focus:ring-offset-2 transition-all text-sm disabled:hover:bg-green-100 disabled:hover:text-green-500"
                    on:click={() => saveSelected()}
                    disabled={unableSaveUpdate}
                  >
                    <Floppy /></button
                  >
                  <button
                    type="button"
                    class="py-2 px-3 inline-block justify-center items-center whitespace-nowrap gap-2 rounded-md bg-orange-100 border border-transparent font-semibold text-orange-500 hover:text-white hover:bg-orange-500 focus:outline-none focus:ring-2 ring-offset-white focus:ring-orange-500 focus:ring-offset-2 transition-all text-sm disabled:hover:bg-orange-100 disabled:hover:text-orange-500"
                    on:click={() => resetSelected()}
                  >
                    <XLg />
                  </button>
                {:else}
                  <button
                    type="button"
                    class="py-2 px-3 inline-block justify-center items-center whitespace-nowrap gap-2 rounded-md bg-gray-100 border border-transparent font-semibold text-gray-500 hover:text-white hover:bg-gray-500 focus:outline-none focus:ring-2 ring-offset-white focus:ring-gray-500 focus:ring-offset-2 transition-all text-sm disabled:hover:bg-gray-100 disabled:hover:text-gray-500"
                    on:click={() => setSelected(id)}
                  >
                    <PencilSquare />
                  </button>

                  <button
                    type="button"
                    class="py-2 px-3 inline-block justify-center items-center whitespace-nowrap gap-2 rounded-md bg-green-100 border border-transparent font-semibold text-green-500 hover:text-white hover:bg-green-500 focus:outline-none focus:ring-2 ring-offset-white focus:ring-green-500 focus:ring-offset-2 transition-all text-sm disabled:hover:bg-green-100 disabled:hover:text-green-500"
                    on:click={() => handleDone(id)}
                  >
                    <CheckSquare />
                  </button>
                {/if}
              </div>
            {/each}
          </div>
        {/if}
      </div>
    </div>

    <!-- part c -->
    <div class="mt-4">
      <h2 class="text-xl font-semibold">Completed task</h2>

      <div>
        {#if completedList.length <= 0}
          <!-- empty state -->
          <div class="w-full text-center">
            <div class="w-full py-8 px-5">
              <img
                src={imgNotionPapers}
                alt="papers"
                class="w-[100px] mx-auto"
              />
            </div>
            <p>No completed task yet.</p>
          </div>
        {:else}
          <ul class="flex flex-col gap-2 mt-4">
            {#each completedList as { id, name }}
              <li class="flex gap-2">
                <span class="block w-full py-2 px-3 border border-transparent">
                  {name}
                </span>
                <button
                  type="button"
                  class="py-2 px-3 inline-block justify-center items-center gap-2 rounded-md border border-transparent font-semibold text-red-500 hover:bg-red-100 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-offset-2 transition-all text-sm"
                  on:click={() => handleDelete(id)}
                >
                  <Trash />
                </button>
              </li>
            {/each}
          </ul>
        {/if}
      </div>
    </div>
  </div>
</section>

<template>
  <div class="min-h-screen bg-[#f0f0f0]">
    <div class="text-center pt-5 pb-[60px]">
      <h1>Event Calendar</h1>
      <div
        id="calendar"
        class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-5 xl:grid-cols-7 gap-[10px] p-5">
        <template v-for="calendarDay in calendarDays" :key="calendarDay.id">
          <div v-if="calendarDay.blank"></div>
          <div
            v-else
            class="flex flex-col items-center bg-white text-[#c4c4c4] p-[15px] rounded-[8px] min-h-[70px] shadow-[0_4px_8px_rgba(0,0,0,0.1)]">
            {{ calendarDay.text }}
            <div
              v-for="task in calendarDay.tasks"
              :key="task.id"
              class="bg-[#212121] text-white p-3 mt-[10px] rounded text-center break-words text-[0.8em] w-[88%] cursor-pointer transition-all duration-300 ease-in-out hover:bg-[#424242]"
              @click="() => editTaskHandler(calendarDay.id, task.id, task.description)"
              @click.right="() => deleteTaskHandler(calendarDay.id, task.id)">
              {{ task.description }}
            </div>
          </div>
        </template>
      </div>
    </div>

    <button
      class="fixed bottom-5 left-1/2 -translate-x-1/2 text-white bg-[#212121] py-[10px] px-12 border-none rounded-[20px] cursor-pointer transition-all ease-in-out duration-300 shadow-[0_4px_8px_rgba(0,0,0,0.2)] hover:bg-[#424242]"
      @click="showAddTaskModalHandler">
      Add Task
    </button>

    <div id="add-task-modal" v-if="showModal" class="fixed z-10 left-0 top-0 w-full h-full overflow-auto bg-black/70">
      <div class="bg-white mx-auto my-[15%] p-5 border border-[#888] border-solid w-[300px] rounded-[14px] text-center">
        <span
          class="text-[#aaa] float-right text-[28px] font-bold cursor-pointer transition-all duration-300 ease-in-out hover:text-black focus:text-black"
          @click="closeAddTaskModalHandler"
          >&times;</span
        >
        <h2>Add Task</h2>
        <input
          type="date"
          id="task-date"
          ref="dateRef"
          class="w-full p-[10px] my-[10px] mx-auto inline-block border border-solid border-[#ccc] rounded-[10px] outline-none" />
        <input
          type="text"
          id="task-desc"
          ref="descRef"
          placeholder="Task Description"
          class="w-full p-[10px] my-[10px] mx-auto inline-block border border-solid border-[#ccc] rounded-[10px] outline-none" />
        <button
          @click="addTaskHandler"
          class="bg-[#212121] text-white py-[10px] px-12 my-2 mx-auto border-none rounded-[14px] cursor-pointer transition-all duration-300 ease-in-out hover:text-[#424242]">
          Add Task
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import { nanoid } from 'nanoid';

export default {
  data() {
    return {
      calendarDays: [],
      showModal: false,
    };
  },
  methods: {
    showAddTaskModalHandler() {
      this.showModal = true;
    },
    closeAddTaskModalHandler() {
      this.showModal = false;
    },
    addTaskHandler() {
      const taskDate = new Date(this.$refs.dateRef.value);
      const taskDesc = this.$refs.descRef.value.trim();
      if (taskDesc && !isNaN(taskDate.getDate())) {
        const tempCalendar = this.calendarDays.map((calendarDay) => {
          if (calendarDay.blank || calendarDay.text !== taskDate.getDate()) {
            return calendarDay;
          } else {
            const tempTasks = calendarDay.tasks;
            tempTasks.push({ id: nanoid(), description: taskDesc });
            return { ...calendarDay, tasks: tempTasks };
          }
        });
        localStorage.setItem('oks-calendar', JSON.stringify(tempCalendar));
        this.calendarDays = tempCalendar;
      } else {
        alert('Please enter a valid date and description!');
      }

      this.closeAddTaskModalHandler();
    },
    deleteTaskHandler(calendarId, taskId) {
      if (confirm('Are you sure you want to delete this task?')) {
        const tempCalendar = this.calendarDays.map((calendarDay) => {
          if (calendarDay.id === calendarId) {
            const tempTask = calendarDay.tasks.filter((task) => task.id !== taskId);
            return { ...calendarDay, tasks: tempTask };
          } else {
            return calendarDay;
          }
        });
        localStorage.setItem('oks-calendar', JSON.stringify(tempCalendar));
        this.calendarDays = tempCalendar;
      }
    },
    editTaskHandler(calendarId, taskId, taskDesc) {
      const newTaskDesc = prompt('Edit your task:', taskDesc);
      if (newTaskDesc !== null && newTaskDesc.trim() !== '') {
        const tempCalendar = this.calendarDays.map((calendarDay) => {
          if (calendarDay.id === calendarId) {
            const tempTasks = calendarDay.tasks.map((task) => {
              if (task.id === taskId) {
                return { ...task, description: newTaskDesc };
              }
              return task;
            });
            return { ...calendarDay, tasks: tempTasks };
          } else {
            return calendarDay;
          }
        });
        localStorage.setItem('oks-calendar', JSON.stringify(tempCalendar));
        this.calendarDays = tempCalendar;
      }
    },
  },
  mounted() {
    if (!localStorage.getItem('oks-calendar')) {
      const currentDate = new Date();
      const month = currentDate.getMonth();
      const year = currentDate.getFullYear();

      const firstDayOfMonth = new Date(year, month, 1);
      const lastDayOfMonth = new Date(year, month + 1, 0);

      const firstDayOfWeek = firstDayOfMonth.getDay();
      const totalDays = lastDayOfMonth.getDate();

      const temCalendars = [];

      for (let i = 0; i < firstDayOfWeek; i++) {
        temCalendars.push({ id: nanoid(), blank: true });
      }

      for (let day = 1; day <= totalDays; day++) {
        temCalendars.push({ id: nanoid(), blank: false, text: day, tasks: [] });
      }
      localStorage.setItem('oks-calendar', JSON.stringify(tempCalendar));
      this.calendarDays = temCalendars;
    } else {
      this.calendarDays = JSON.parse(localStorage.getItem('oks-calendar'));
    }
  },
};
</script>

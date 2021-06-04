<template>
    <AddTask @add-task="addTask" v-show="showAddTask" ></AddTask>
    <Tasks :tasks="tasks" 
            @delete-task="deleteTask"
            @toggle-reminder="toggleReminder">
    </Tasks>
</template>

<script>
import Tasks from '../components/Tasks'
import AddTask from '../components/AddTask'

export default {
    name: 'Home',
    components: {
        Tasks,
        AddTask
    },
    props: {
        showAddTask: Boolean
    },
    data() {
        return {
            tasks: []
        }
    },
    methods: {
		async deleteTask(id) {
			if(confirm('Are you sure?')) {
				const res = await fetch(`api/tasks/${id}`, {
					method: 'DELETE',
					headers: {'Content-type': 'application/json'},
				});

				if(res.status === 200) {
					this.tasks = this.tasks.filter(
						(task) => task.id !== id
					) 
				} else 
					alert('Error deleting task')
			}
		},
		async toggleReminder(id) {
			const targetTask = await this.fetchTask(id);
			const updTask = {...targetTask, reminder: !targetTask.reminder}
			const res = await fetch(`api/tasks/${id}`, {
				method: 'PUT',
				headers: {'Content-type': 'application/json'},
				body: JSON.stringify(updTask)
			})
			
			const data = await res.json();
			this.tasks = this.tasks.map(
				(task) => task.id === id 
					? {...task, reminder: data.reminder} : task
			)
		
		},
		async addTask(task) {
			const res = await fetch('api/tasks', {
				method: 'POST',
				headers: {'Content-type': 'application/json'},
				body: JSON.stringify(task)
			});

			const data = await res.json();
			this.tasks.push(data)
		},
		async fetchTasks() {
			const res = await fetch('api/tasks')
			const data = await res.json();
			return data;
		},
		async fetchTask(id) {
			const res = await fetch(`api/tasks/${id}`)
			const data = await res.json();
			return data;
		}
	},
    async created() {
		this.tasks = await this.fetchTasks()
	}
}
</script>

<style scoped>

</style>
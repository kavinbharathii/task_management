<script>
import Project from '../components/Project.vue'
import { ref, get, child, onChildChanged, update, set } from "firebase/database"
import { db , auth  } from "../firebase"
import Router from '../router/index'
import { onAuthStateChanged } from 'firebase/auth'

export default {
    name: "HomeView",
	components: {
		Project
	},  
	data() {
		return {
			projects: [],
			newProjectName: ""
		}
	},
	methods: {
		createNewProject() {
			if (this.newProjectName !== "") {
				console.log(this.newProjectName)
				this.projects.push({
					name: this.newProjectName,
				})
			}
            this.addProjectToDatabase()
			this.newProjectName = ""
		},
		async addProjectToDatabase() {
			await set(ref(db, this.newProjectName), {
				name: this.newProjectName
			});
		},

		getuser() {
			onAuthStateChanged(auth, (user) => {
				if (user) {
					const uid = user.email;
					this.email = uid.split('@')[0]
				} else {
					console.log("Can't get user e-mail")
					const route = useRoute()
					Router.push('/signin')
				}
			});
		}
	},
    created() {
        let fetchedProjects = get(ref(db)).then((snapshot) => {
            if (snapshot.exists()) {
                for (let val in snapshot.val()) {
                    this.projects.push({
                        name: val,
                    })
                }
            } else {
                console.log("No data available")
            }
        }).catch((error) => {
            console.log(error)
        })
    },
	mounted: function() {
		this.getuser()
	}
}
</script>

<template>
	<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:opsz,wght,FILL,GRAD@48,400,0,0" />
	<div id="index">
		<div style="font-size: 35px;display: flex;align-items: center;justify-content: center; margin-bottom: 1em">Projects</div>
		<Project v-for="(proj,ind) in projects" :key="ind" :name = proj.name :created= proj.created />
	
		<form @submit.prevent="this.createNewProject" class="project-form">
			<input type="text" v-model="newProjectName" class="project">
			<button class="pro-btn"><span class="material-symbols-outlined" style="color: white">add</span></button>
		</form>
	</div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Poppins&display=swap');
input {
	width: 50vw;
    padding: .5em;
	outline: none;
	border: none;
	border-radius: 10px;
	background: #171717;
	height: 2rem;
	color: #dedede;
	font-family: 'Poppins', sans-serif;
}

.project-form {
	/* width: 100vw; */
	/* margin: 1rem 5rem; */
	border-radius: 10px;
	color: #dedede;
	margin-top: 1.5rem;
	margin-left: 2rem;
}

button {
	background-color: #242424;
    border: none;
    outline: none;
	position: relative;
	left: -4rem;
	top: 0.5rem;
	background-color: #171717;
}

#index {
	overflow: hidden;
	width: 100vw;
	font-family: 'Poppins', sans-serif;

	display: flex;
	justify-content: center;
	align-items: center;
	flex-direction: column;
}

</style>

<template>
    <div class="w-25">
        <form @submit.prevent="saveData">
            <div class="input-group mb-3 w-100">
            <input v-model="form.title" :class="{'is-invalid' : form.errors.has('title')}" type="text" class="form-control form-control-lg"  @keydown="form.errors.clear('title')"
            aria-label="Recipient's username" aria-describedby="button-addon2">
            <div class="input-group-append">
                <button class="btn btn-success" type="submit" id="button-addon2">Add this to your list</button>
            </div>
            </div>
            <span class="text-danger pt-3 pb-3" style="font-size:20px;" v-if="form.errors.has('title')" v-text="form.errors.get('title')"></span>
        </form>
        <div class="w-100 todo">
            <div v-for="todo in todos" :key="todo.id" class="w-100 d-flex align-items-center p-3 bg-white border-bottom">
                <span class="mr-2">
                <svg v-on:click="toggleTodo(todo)" v-if="todo.completed == false" xmlns="http://www.w3.org/2000/svg" class="icon icon-tabler icon-tabler-circle" width="36" height="36" viewBox="0 0 24 24" stroke-width="1.5" stroke="#FFC107" fill="none" stroke-linecap="round" stroke-linejoin="round">
                    <path stroke="none" d="M0 0h24v24H0z"/>
                    <circle cx="12" cy="12" r="9" />
            </svg> 
            <svg v-if="todo.completed == true" v-on:click="toggleTodo(todo)" xmlns="http://www.w3.org/2000/svg" class="icon icon-tabler icon-tabler-circle-check" width="36" height="36" viewBox="0 0 24 24" stroke-width="1.5" stroke="#4CAF50" fill="none" stroke-linecap="round" stroke-linejoin="round">
            <path stroke="none" d="M0 0h24v24H0z"/>
            <circle cx="12" cy="12" r="9" />
            <path d="M9 12l2 2l4 -4" />
            </svg>
            </span>

            <div  class="font-weight-bolder"><span v-if="editmode == false || editmode != todo.id">{{todo.title}}</span><input v-if="editmode == todo.id"   v-model="todo.title" type="text">
            </div>

            <div class="ml-auto mr-2 d-flex align-items-center"><span>
    
    <svg v-on:click="editmode = todo.id"  v-if="editmode != todo.id" xmlns="http://www.w3.org/2000/svg" class="icon icon-tabler icon-tabler-edit" width="36" height="36" viewBox="0 0 24 24" stroke-width="1.5" stroke="#FFC107" fill="none" stroke-linecap="round" stroke-linejoin="round">
  <path stroke="none" d="M0 0h24v24H0z"/>
  <path d="M9 7 h-3a2 2 0 0 0 -2 2v9a2 2 0 0 0 2 2h9a2 2 0 0 0 2 -2v-3" />
  <path d="M9 15h3l8.5 -8.5a1.5 1.5 0 0 0 -3 -3l-8.5 8.5v3" />
  <line x1="16" y1="5" x2="19" y2="8" />
</svg>
<svg v-if="editmode == todo.id" v-on:click="updateTodo(todo)"  xmlns="http://www.w3.org/2000/svg" class="icon icon-tabler icon-tabler-checkbox" width="36" height="36" viewBox="0 0 24 24" stroke-width="1.5" stroke="#4CAF50" fill="none" stroke-linecap="round" stroke-linejoin="round">
  <path stroke="none" d="M0 0h24v24H0z"/>
  <polyline points="9 11 12 14 20 6" />
  <path d="M20 12v6a2 2 0 0 1 -2 2h-12a2 2 0 0 1 -2 -2v-12a2 2 0 0 1 2 -2h9" />
</svg>
</span>
<span>
    <svg v-on:click="deleteTodo(todo)" xmlns="http://www.w3.org/2000/svg" class="icon icon-tabler icon-tabler-trash ml-1" width="36" height="36" viewBox="0 0 24 24" stroke-width="1.5" stroke="#FF5722" fill="none" stroke-linecap="round" stroke-linejoin="round">
  <path stroke="none" d="M0 0h24v24H0z"/>
  <line x1="4" y1="7" x2="20" y2="7" />
  <line x1="10" y1="11" x2="10" y2="17" />
  <line x1="14" y1="11" x2="14" y2="17" />
  <path d="M5 7l1 12a2 2 0 0 0 2 2h8a2 2 0 0 0 2 -2l1 -12" />
  <path d="M9 7v-3a1 1 0 0 1 1 -1h4a1 1 0 0 1 1 1v3" />
</svg></span>
</div>

            </div>
        </div>
    </div>
</template>

<script>
    export default {
        data(){
            return{
                editmode: false,
                todos:'',
                form: new Form({
                    title: '',
                })
            }
        },
        methods:{
            deleteTodo(e){
                let data = new FormData();
                data.append('_method', 'DELETE')
                axios.post('/api/todo/'+e.id, data).then((res) =>{
                    this.todos = res.data
                }).catch((error) => {
                    this.form.errors.record(error.response.data.errors)
                })
            },
            updateTodo(e){
                this.editmode = false
                let data = new FormData();
                data.append('_method', 'PATCH')
                data.append('title', e.title)
                axios.post('/api/todo/'+e.id, data)
                .catch((error) => {
                    this.form.errors.record(error.response.data.errors)
                })
            },

            toggleTodo(e){
                e.completed = !e.completed
                let data = new FormData();
                data.append('_method', 'PATCH')
                if(e.completed == true){
                    data.append('completed', 1);
                }
                if(e.completed == false){
                    data.append('completed', 0)
                }
                axios.post('/api/todo/'+e.id, data)
            },
            getTodos(){
                    axios.get('/api/todo').then((res) =>{
                        this.todos = res.data
                    }).catch((error) =>{
                        console.log(error)
                    })
            },

            saveData(){
                let data = new FormData();
                data.append('title', this.form.title)
                axios.post('/api/todo', data).then((res) =>{
                    this.form.reset()
                     this.getTodos()
                }).catch((error) => {
                    this.form.errors.record(error.response.data.errors)
                })
            }
        },

        mounted() {
          this.getTodos()
        }
    }
</script>

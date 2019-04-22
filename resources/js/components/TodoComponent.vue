<template>
    <div class="container">
        <div class="row text-center">
            <div class="col-12 text-center mt-5">
                <h1>Todo List</h1>
                <h5 class="mt-4"><a href="" @click.prevent="showForm();" class="my-4">Add Todo</a></h5>
            </div>
            <transition name="fade">
            <div class="col-12" :key="loading" >
                <div class="card mt-4" v-for="todo in todos" :key="todo.id">
                    <div class="card-body">
                        <h5 class="card-title">{{todo.title}}</h5>
                        <p class="card-text">{{todo.desc}}</p>
                        <a href="" @click.prevent="showForm(todo);edit=true" class="card-link">Edit</a>
                        <a href="" @click.prevent="deleteTodo(todo)" class="card-link">Delete</a>
                    </div>
                </div>
            </div>
            </transition>
        </div>

        <modal name="form" :height="'auto'">
            <div class="row text-center  p-5">
                <div class="col-12  ">
                    <h3>{{ edit ? 'Edit Todo' :'Add Todo'}}</h3>
                    <div class="form-group my-3">
                        <label>Title</label>
                        <input v-model.trim="todo.title" type="text" class="form-control" id="exampleInputEmail1" aria-describedby="emailHelp" placeholder="Enter title of your todo">
                    </div>
                    <div class="form-group my-3">
                        <label>Desc</label>
                        <input v-model.trim="todo.desc" type="text" class="form-control" placeholder="Enter description for todo , eg: Place or reminder">
                    </div>
                </div>
                <div class="col-12 my-3 ">
                    <button  @click.prevent="hide" type="button" class="btn btn-danger">Cancel</button>
                    <button  :disabled="isButtonDisabled" @click.prevent="submitTodo" type="button" class="btn btn-primary">Submit</button>
                </div>
            </div>
        </modal>
    </div>

</template>

<script>
    import 'sweetalert';
    export default {
        methods: {
            showForm (todo) {                
                this.$modal.show('form');
                todo ? this.todo = JSON.parse(JSON.stringify(todo)) : Object.keys(this.todo).forEach(key=>{
                    this.todo[key] = null;
                })
            },
            hide () {
                this.$modal.hide('form');
                this.edit = false;
            },
            deleteTodo(item){
                swal({
                    title: 'Warning!',
                    text: 'Are you sure ?',
                    icon: "warning",
                    buttons: {
                        cancel: "Cancel",
                        confirm: {
                            text: "Yes, delete",
                            value: true,
                            closeModal: false,
                        }
                    },
                    dangerMode: true,
                })
                .then(val => {
                    if (val == null) return null
                    return axios.post(`/api/todos/${item.id}`,{
                        _method:'delete',
                    });
                })
                .then(results => {
                    if(results){
                        console.log(results);
                        swal({
                            icon:"success",
                            title: "Success!",
                            text: "Successfully delete todos",
                        });
                        this.loadData();   
                    }
                    
                })
                .catch(err=>{
                    console.log(err)
                })
            },
            submitTodo(){
                this.isButtonDisabled = true;
                let url;
                if(this.edit){
                    this.todo._method = 'patch';
                    url = `/api/todos/${this.todo.id}` 
                } else {
                    url = '/api/todos'
                }
                axios.post(url,this.todo).then(res=>{
                    this.isButtonDisabled = false;
                    if (this.edit) {
                        swal("Success!", "Sucessfuly edited todo!", "success");
                    } else {
                        swal("Success!", "Sucessfuly creted todo!", "success");
                    }
                    this.loadData();
                    this.hide();

                })
                .catch(err=>{
                    console.log(err);
                })
            },
            loadData(){
                this.loading = true;
                this.todos.length = 0;
                axios.get('/api/todos')
                .then(res=>{
                    this.todos = res.data;
                    this.loading = false;
                })
            }
        },
        data(){
            return{
                edit:false,
                todos:[],
                todo:{
                    id:null,
                    title:null,
                    desc:null
                },
                isButtonDisabled:false,
                loading:true
            }
        },
        mounted(){
            this.loadData();
        }
    }
</script>

<style>
html, body {
  height:100%;
}
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}

</style>


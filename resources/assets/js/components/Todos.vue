<style scoped>
.done {
   text-decoration: line-through;
}
</style>

<template>
    <div class="main">

        <!-- .........................................
                        Todo_form 
        ............................................-->
        <section id="Todo_form">
            <div class="modal fade" id="exampleModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
                <div class="modal-dialog" role="document">
                    <div class="modal-content">

                        <form @submit.prevent>
                            <div class="modal-header">
                                <!-- <h5 class="modal-title" id="exampleModalLabel">Modal title</h5> -->

                                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                                    <span aria-hidden="true">&times;</span>
                                </button>
                            </div>

                            <div class="modal-body">
                                <div class="form-group">
                                    <input type="text" v-model="todo.title" id="todo_title" class="form-control" placeholder="Task">
                                </div>
                                <div class="form-group">
                                    <textarea id="todo_body" class="form-control"  v-model="todo.body" rows="10" placeholder="Description"></textarea>
                                </div>
                            </div>

                            <div class="modal-footer">
                                <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                                <button type="button" @click="addTodo()" class="btn btn-primary">Save changes</button>
                            </div>
                        </form>

                    </div>
                </div>
            </div>
        </section>
            
        <br>
        <br>
        <br>

        <!-- .........................................
                        navigation 
        ............................................-->
        <section id="navigation">
            <nav aria-label="Page navigation ">
                <span class="btn-group-sm">
                    <button type="button" class="btn btn-danger bmd-btn-fab"  data-toggle="modal" data-target="#exampleModal"
                    @click="clearForm()">
                        <i class="material-icons"> add </i>
                    </button>
                </span>

                <ul class="pagination justify-content-end">
                    <li v-bind:class="[{disabled : !pagination.prev_page_url}]" class="page-item">
                        <a class="page-link" href="#" tabindex="-1"   @click="fetchTodos(pagination.prev_page_url)">Previous</a>
                    </li>

                    <li class="page-item disabled">
                        <a class="page-link" href="#"> Page {{ pagination.current_page }} of {{ pagination.last_page }} </a>
                    </li>

                    <li  v-bind:class="[{disabled : !pagination.next_page_url}]"  class="page-item">
                        <a class="page-link" href="#"   @click="fetchTodos(pagination.next_page_url)">Next</a>
                    </li>
                </ul>
            </nav>
        </section>

        <div class="card-columns">
            <div class="card border-secondary "  v-for="todo in todos" v-bind:key="todo.id">

                <div  v-bind:class="[{'text-muted ' : todo.done}]" class="card-body" >

                    <div v-bind:class="[{'done ' : todo.done}]">
                        <!-- Title -->
                        <h5 class="card-title">{{ todo.title }}</h5>

                        <!-- Body -->
                        <p class="card-text" > {{ todo.body }}</p>
                    </div>

                    <!-- Update to -->
                    <div class="card-text">
                        <small class="text-muted">
                            {{ todo.updated_at  | moment(" MMMM DD.YYYY") }}
                        </small>

                        <button type="button" v-bind:class="[{'btn-success' : !todo.done, 'btn-danget' : todo.done}]" class="btn  small"  @click="editTodo(todo, !todo.done);addTodo();">

                            <i class="material-icons">check</i>
                        </button>
                        
                        <button type="button" class="btn btn-warning small" data-toggle="modal" data-target="#exampleModal" 
                             @click="editTodo(todo, todo.done)">

                            <i class="material-icons">edit</i>
                        </button>

                        <button type="button" class="btn btn-danger" 
                            @click="deleteTodo(todo.id)">

                            <i class="material-icons"> delete </i>
                        </button>
                    </div>
                
                </div>

            </div>
        </div>

    </div>
</template>

<script>
export default {
    data(){
        return {
            todos: [],
            todo: {
                id: '',
                title: '',
                body: '',
                todo_id: '',
                done: false
            },
            pagination: {},
            edit: false

        }
    },

    created(){
        this.fetchTodos();
    },

    methods: {
        fetchTodos(page_url){
            let vm = this;
            page_url = page_url || 'api/todos'; 
            fetch(page_url)
            .then(res => res.json())
            .then(res => {
                this.todos = res.data;
                vm.makePagination(res.meta, res.links);

            })
            .catch(err => console.log(err));
        },
        makePagination(meta, links){
            let pagination = {
                current_page: meta.current_page,
                last_page: meta.last_page,
                next_page_url: links.next,
                prev_page_url: links.prev
            }

            this.pagination = pagination;
        },
        deleteTodo(id){
            if(confirm('Are you Sure?')){
                fetch(`api/todo/${id}`, {
                    method: 'delete',
                })
                .then(res => res.json())
                .then(data => {
                    alert('Todo is removed');
                    this.fetchTodos();
                })
                .catch(err => console.log(err));
            }
        },
        addTodo(){
            
            if(this.edit === false){
                //Add
                delete this.todo.todo_id;
                fetch('api/todo', {
                    method: 'post',
                    body: JSON.stringify(this.todo),
                    headers: {
                        'content-type': 'application/json'
                    }
                })
                .then(res => res.json())
                .then(data => {
                    this.todo.title = '';
                    this.todo.body = '';
                    this.todo.done = false;
                    this.edit = false;

                    this.fetchTodos();

                    alert('Todo added');

                    $('.close').click();
                    $('.modal-backdrop').hide();

                })
                .catch(err => console.log(err));

     
            }else {
                //Edit
                console.log(JSON.stringify(this.todo));
                fetch('api/todo', {
                    method: 'put',
                    body: JSON.stringify(this.todo),
                    headers: {
                        'content-type': 'application/json'
                    }
                })
                .then(res => res.json())
                .then(data => {
                    this.todo.title = '';
                    this.todo.body = '';
                    this.todo.done = false;
                    this.edit = false;

                    $('.close').click();
                    $('.modal-backdrop').hide();

                    this.fetchTodos();
                    
                    alert('Todo updated');
                    
                })
                .catch(err => console.log(err));
            }
 
        },
        editTodo(todo, check){
            this.edit = true;
            this.todo.id = todo.id;
            this.todo.todo_id = todo.id;
            this.todo.title = todo.title;
            this.todo.body = todo.body;
            this.todo.done = check;
        },
        clearForm(){
            this.edit = false;
            this.todo.id = "";
            this.todo.todo_id = "";
            this.todo.title = "";
            this.todo.body = "";
            this.todo.done = false;
        }
    }

}
</script>


<template>
    <div class="container">
        <div class="row mt-5">
          <div class="col-12">
            <div class="card" v-if="$gate.isAuth()" >
              <div class="card-header">
                <h3 class="card-title">Users Table</h3>
                <div class="card-tools">
                    <button class="btn btn-success" data-toggle="modal" @click="newModal">Add New</button>
                </div>
            
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <tbody><tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th>Email</th>
                    <th>Type</th>
                    <th>Registred At</th>
                    <th>Modify</th>
                  </tr>
                  <tr v-for="user in users.data" :key="user.id">
                    <td>{{user.id}}</td>
                    <td>{{user.name}}</td>
                    <td>{{user.email}}</td>
                    <td>{{user.type | upText}}</td>
                    <td>{{user.created_at }}</td>
                    <td v-if="$gate.idUser() === user.id">
                        <a href="#" @click="editModal(user)">
                            <i class="fa fa-edit text-blue"></i>
                        </a>
                        /
                        <a href="#" @click="deleteUser(user.id)">
                            <i class="fa fa-trash text-red"></i>
                        </a>
                       </td>
                  </tr>
                  
                </tbody></table>
              </div>
              <!-- /.card-body -->
             <div class="card-footer">
               <pagination :data="users"  @pagination-change-page="getResults"></pagination>
             </div> 
            </div>
            <!-- /.card -->
          </div>
        </div>
            <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
              <div class="modal-dialog " role="document">
                <div class="modal-content">
                  <div class="modal-header">
                    <h5 v-show="!edit" class="modal-title" id="exampleModalLabel">Add New</h5>
                    <h5 v-show="edit" class="modal-title" id="exampleModalLabel">Update</h5>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                      <span aria-hidden="true">&times;</span>
                    </button>
                  </div>
                  <form @submit.prevent="edit ? updateUser() : createUser()">
                      <div class="modal-body">
                         <div class="form-group">
                            <input v-model="form.name" type="text" name="name" id="name" placeholder="name" 
                              class="form-control" :class="{'is-invalid': form.errors.has('name')}">
                              <has-error :form="form" field="name"></has-error>
                        </div>
                        <div class="form-group">
                            <input v-model="form.email" type="email" name="email" id="email" placeholder="email" 
                              class="form-control" :class="{'is-invalid': form.errors.has('email')}">
                              <has-error :form="form" field="email"></has-error>
                        </div>
                     
                      <div class="form-group">
                            <textarea v-model="form.bio" type="text" name="bio" id="bio"
                              class="form-control" :class="{'is-invalid': form.errors.has('bio')}"></textarea>
                              <has-error :form="form" field="bio"></has-error>
                        </div>
                        <div class="form-group">
                            <select v-model="form.type"name="type" id="type"
                              class="form-control" :class="{'is-invalid': form.errors.has('type')}">
                              <option value="">Select User Role</option>
                              <option value="admin">Admin</option>
                              <option value="user">User</option>
                              <option value="author">Author</option>
                            </select> 
                              <has-error :form="form" field="type"></has-error>
                        </div> 
                        <div class="form-group">
                            <input v-model="form.password" type="password" name="password" id="password"
                              class="form-control" :class="{'is-invalid': form.errors.has('password')}">
                              <has-error :form="form" field="password"></has-error>
                        </div>
                      </div>
                      <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                        <button v-show="!edit" type="submit" class="btn btn-primary">Create</button>
                        <button  v-show="edit" type="submit" class="btn btn-primary">Update</button>
                      </div>
                 </form>
                </div>
              </div>
            </div>

    </div>


</template>

<script>
    export default {

        data(){
          return {
            edit: false,
            users: {},
            form: new form({
                id: '',
                name: '',
                email: '',
                password: '',
                type: '',
                bio: ''               
              
            })
          }
        },
        methods: {
            loadUsers(){
               axios.get('api/user')
                    .then(({data}) => (this.users = data));
            },
             getResults(page = 1) {
                     axios.get('api/user?page=' + page)
                           .then(response => {
                             this.users = response.data;
                      });
              },
            createUser(){
                this.$Progress.start()
                this.form.post('api/user')
                 .then(()=>{
                     Fire.$emit('AfterCreate');
                    $('#addNew').modal('hide');
                    toast.fire({
                          type: 'success',
                          title: 'User created successfully'
                        })
                    this.$Progress.finish();
                 })
                .catch(()=>{

                });
               
            },
            deleteUser(id){
               swal.fire({
                      title: 'Are you sure?',
                      text: "You won't be able to revert this!",
                      type: 'warning',
                      showCancelButton: true,
                      confirmButtonColor: '#3085d6',
                      cancelButtonColor: '#d33',
                      confirmButtonText: 'Yes, delete it!'
                    }).then((result) => {
                     if (result.value) {
                          this.form.delete('api/user/' + id).then(()=>{
                                
                              swal.fire(
                              'Deleted!',
                              'Your file has been deleted.',
                              'success'
                              );
                            Fire.$emit('AfterCreate');
                          }).catch(()=>{
                          swal('Failed', 'There was something wrong', 'warning')
                        });
                          } 
                        
                        });
                
            },
            newModal(){
              this.edit = false;
              this.form.reset();
              this.form.clear(); //funkcija iz v-form
              $('#addNew').modal('show');
            },
            editModal(user){
              this.edit = true;
              this.form.reset();
              this.form.clear(); //funkcija iz v-form
              $('#addNew').modal('show');
              this.form.fill(user);
            },
            updateUser(){
             this.$Progress.start();

             this.form.put('api/user/' + this.form.id)
             .then(()=>{
               $('#addNew').modal('hide');
                swal.fire(
                'Updated!',
                'Your file has been updated.',
                'success'
                 );
                Fire.$emit('AfterCreate');
                this.$Progress.finish();
             })
             .catch(()=>{
              this.$Progress.fail();
             });
            },
        },
        created() {
            Fire.$on('searching',() => {
                let query = this.$parent.search;
                axios.get('api/findUsers?q=' + query)
                .then((data) => {
                    this.users = data.data
                })
               .catch(()=>{

               })
             });
         
             this.loadUsers();
             Fire.$on('AfterCreate', () =>{
                this.loadUsers();
             });
        }
    }
</script>

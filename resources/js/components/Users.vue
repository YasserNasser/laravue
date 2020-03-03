<template>
    <div class="container ">
    <div class="row " v-if="$gate.isAdminOrAuthor()">
        <div class="col-12 mt-4">
            <div class="card" >
                <div class="card-header">
                    <h3 class="card-title">Users Table</h3>
                    <div class="card-tools">
                        <button class="btn btn-success " @click="createModal()">Create User <i class="fa fa-user-plus fa-fw"></i></button>
                    </div>

                </div>

                <!-- /.card-header -->
                <div class="card-body table-responsive p-0 ">
                    <table class="table table-hover">
                        <thead>
                        <tr>
                            <th>ID</th>
                            <th>Name</th>
                            <th>Email</th>
                            <th>Type</th>
                            <th>Created At</th>
                            <th>Action</th>
                        </tr>
                        </thead>
                        <tbody>
                        <tr v-for="user in users.data" :key="user.id">
                            <td>{{user.id}}</td>
                            <td>{{user.name}}</td>
                            <td>{{user.email}}</td>
                            <td><span class="tag tag-success">{{user.type | upText}}</span></td>
                            <td>{{user.created_at | formatDate}}</td>
                            <td><a @click="editModal(user)">
                                <i class="fa fa-edit green"></i>
                            </a> |
                                <a  @click="deleteUser(user.id)">
                                    <i class="fa fa-trash red"></i>
                                </a>
                            </td>
                        </tr>

                        </tbody>
                    </table>
                </div>
                <!-- /.card-body -->
                <div class="card-footer"><pagination :data="users" @pagination-change-page="getResults"></pagination></div>
            </div>
            <!-- /.card -->
        </div>
    </div>

        <div class="row " v-if="!$gate.isAdminOrAuthor()">
            <not-found></not-found>
        </div>

        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNewLabel" aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" v-show="editMode" id="addNewLabel">Update User</h5>
                        <h5 class="modal-title" v-show="!editMode" id="addNewLabel">Add New User</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <form @submit.prevent="editMode ? updateUser(user) : createUser()">
                    <div class="modal-body">
                        <div class="form-group">
                            <input v-model="form.name" type="text" name="name"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('name') }"
                            placeholder="User Name">
                            <has-error :form="form" field="name"></has-error>
                        </div>
                        <div class="form-group">
                            <input v-model="form.email" type="email" name="email"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('email') }"
                                   placeholder="Email">
                            <has-error :form="form" field="email"></has-error>
                        </div>
                        <div class="form-group">
                            <input v-model="form.password" type="password" name="password"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('password') }"
                                   placeholder="Password">
                            <has-error :form="form" field="password"></has-error>
                        </div>
                        <div class="form-group">
                            <textarea v-model="form.bio" type="text" name="bio"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"
                                      placeholder="About Me"></textarea>
                            <has-error :form="form" field="bio"></has-error>
                        </div>
                        <div class="form-group">
                            <select v-model="form.type" type="password" name="type"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('type') }"
                                    >
                                <option value="">Select User Type</option>
                                <option value="admin">Admin</option>
                                <option value="user">Stander User</option>
                                <option value="author">Author</option>
                            </select>
                            <has-error :form="form" field="type"></has-error>
                        </div>
                        <div class="form-group">
                            <input @change="Uphoto" type="file" name="photo"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('photo') }"
                                   >
                            <has-error :form="form" field="photo"></has-error>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                        <button type="submit" v-show="editMode" class="btn btn-primary">Update</button>
                        <button type="submit" v-show="!editMode" class="btn btn-primary">Create</button>
                    </div>
                </form>
                </div>
            </div>
        </div>
    </div>

</template>

<script>
    export default {
        data() {
            return {
                editMode: false,
                users:{},
                form: new form({
                    id:'',
                    name: '',
                    email: '',
                    password: '',
                    type: '',
                    bio: '',
                    photo: ''
                })
            }
        },
        methods: {
            getResults(page = 1) {
                axios.get('api/user?page=' + page)
                    .then(response => {
                        this.users = response.data;
                    });
            },
            Uphoto(){

            },
            updateUser(){
                this.$Progress.start()
              this.form.put('api/user/'+this.form.id)
                  .then(() => {
                      this.$Progress.finish()
                      Fire.$emit('AfterCreated');
                      $('#addNew').modal('hide');
                      toast.fire({
                          icon: 'success',
                          title: 'The User Updated successfully'
                      })
                  })
                  .catch(() => {
                      this.$Progress.fail()
                      toast.fire({
                          icon: 'warning',
                          title: 'The User Not Updated successfully'
                      })
                  });
            },
            createModal(){
                this.editMode= false;
                this.form.reset();
                $('#addNew').modal('show');
            },
            editModal(user){
                this.editMode= true;
                this.form.reset();
                $('#addNew').modal('show');
                this.form.fill(user);
            },
            deleteUser(id){
                this.$Progress.start()
                swal.fire({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    icon: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                }).then((result) => {
                    if(result.value) {
                        this.form.delete('api/user/' + id).then(() => {
                            Fire.$emit('AfterCreated');

                            swal.fire(
                                'Deleted!',
                                'User has been deleted.',
                                'success'
                            )
                            this.$Progress.finish()

                        }).catch(() => {
                            this.$Progress.fail()
                        });
                    }
                })
            },
            loadUsers(){
                if(this.$gate.isAdminOrAuthor()) {


                    this.$Progress.start();
                    axios.get("api/user").then(({data}) => (this.users = data));

                    this.$Progress.finish();
                }
            },
          createUser(){
              this.$Progress.start();
              this.form.post('api/user')
                  .then(() =>{
                      $('#addNew').modal('hide');
                      //this.loadUsers();
                      Fire.$emit('AfterCreated');
                      toast.fire({
                          icon: 'success',
                          title: 'The User Created successfully'
                      })
                      this.$Progress.finish();
                  })
                  .catch(() =>{
                      this.$Progress.fail();
                  })

          }
        },
        created() {
            //console.log('Component mounted.')
            this.loadUsers();
            Fire.$on('AfterCreated',() =>{
               this.loadUsers();
            });
        }
    }
</script>

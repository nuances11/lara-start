<template>
    <div class="container">
        <div class="row mt-3" v-show="$gate.isAdminOrAuthor()">
            <div class="col-12">
                <div class="card">
                    <div class="card-header">
                        <h3 class="card-title">Users Table</h3>

                        <div class="card-tools">
                            <button class="btn btn-success" @click="newModal"><i
                                    class="fas fa-user-plus"></i> Add New User</button>
                        </div>
                    </div>
                    <!-- /.card-header -->
                    <div class="card-body table-responsive p-0">
                        <table class="table table-hover">
                            <tbody>
                                <tr>
                                    <th>ID</th>
                                    <th>Name</th>
                                    <th>Email</th>
                                    <th>Type</th>
                                    <th>Registered Date</th>
                                    <th>Modify</th>
                                </tr>

                                <tr v-for="user in users.data" :key="user.id">
                                    <td>{{ user.id }}</td>
                                    <td>{{ user.name }}</td>
                                    <td>{{ user.email }}</td>
                                    <td>{{ user.type | upText }}</td>
                                    <td>{{ user.created_at | readableDate }}</td> 
                                    <td>
                                        <a href="#" @click="editModal(user)">
                                            <i class="fas fa-edit text-success"></i>
                                        </a>
                                        |
                                        <a href="#" @click="deleteUser(user.id)">
                                            <i class="fas fa-trash-alt text-danger"></i>
                                        </a>
                                    </td>
                                </tr>

                                <tr v-show="usersEmpty">
                                   <td colspan="6" class="text-center">No result found!</td> 
                                </tr>

                            </tbody>
                        </table>
                    </div>
                    <!-- /.card-body -->
                    <div class="card-footer">
                        <pagination :data="users" @pagination-change-page="getResults"></pagination>
                    </div>
                </div>
                <!-- /.card -->
            </div>
        </div>

        <div v-show="!$gate.isAdminOrAuthor()">
            <page-not-found></page-not-found>
        </div>

        <!-- Modal -->
        <div class="modal fade" id="addNewUser" tabindex="-1" role="dialog" aria-labelledby="addNewUserLabel"
            aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 v-show="!editmode" class="modal-title" id="addNewUserLabel">Add New</h5>
                        <h5 v-show="editmode" class="modal-title" id="addNewUserLabel">Edit User</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <form @submit.prevent="editmode ? updateUser() : createUser()">
                    <div class="modal-body">
                        <div class="form-group">
                            <label for="name">Name</label>
                            <input v-model="form.name" type="text" class="form-control" name="name" id="name"
                                placeholder="Enter Name" :class="{ 'is-invalid' : form.errors.has('name')}">
                                <has-error :form="form" field="name"></has-error>
                        </div>

                        <div class="form-group">
                            <label for="email">Email</label>
                            <input v-model="form.email" type="email" class="form-control" name="email" id="email"
                                placeholder="ex : sample@sample.com" :class="{ 'is-invalid' : form.errors.has('email')}">
                                <has-error :form="form" field="email"></has-error>
                        </div>

                        <div class="form-group">
                            <label for="bio">Bio (Optional)</label>
                            <textarea v-model="form.bio" name="bio" id="bio" cols="30" rows="10" class="form-control" placeholder="Say something about yourself" :class="{ 'is-invalid' : form.errors.has('bio')}"></textarea>
                            <has-error :form="form" field="bio"></has-error>
                        </div>

                        <div class="form-group">
                            <label for="type">Type</label>
                            <select v-model="form.type" name="type" id="type" class="form-control" :class="{ 'is-invalid' : form.errors.has('type')}">
                                <option value="">Select User Role</option>
                                <option value="admin">Admin</option>
                                <option value="user">Standard User</option>
                                <option value="author">Author</option>
                            </select>
                            <has-error :form="form" field="type"></has-error>
                        </div>
                        
                        <div class="form-group">
                            <label for="password">Password</label>
                            <input v-model="form.password" type="password" class="form-control" name="password" id="password" :class="{ 'is-invalid' : form.errors.has('password')}">
                            <has-error :form="form" field="password"></has-error>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                        <button v-show="editmode" type="submit" class="btn btn-warning">Update</button>
                        <button v-show="!editmode" type="submit" class="btn btn-primary">Create</button>
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
                editmode : false,
                users : {}, 
                form: new Form({
                    id: '',
                    name: '',
                    email: '',
                    password: '',
                    type: '',
                    bio: '',
                    photo: '',
                })
            }
        },
        methods: {
            usersEmpty(){
                let is_empty = (Object.keys(this.users).length === 0) ? true : false;
                console.log(is_empty);
                // return (Object.keys(this.users).length === 0) ? true : false;
            },
            // Our method to GET results from a Laravel endpoint
            getResults(page = 1) {
                axios.get('api/user?page=' + page)
                    .then(response => {
                        this.users = response.data;
                    });
            },
            updateUser(id){
                this.$Progress.start();
                this.form.put('api/user/' + this.form.id)
                .then(() => {
                    Fire.$emit('AfterCreate');
                    $('#addNewUser').modal('hide');
                    Toast.fire({
                        type: 'success',
                        title: 'User updated successfully'
                    })
                    this.$Progress.finish()
                })
                .catch(() => {
                    Toast.fire({
                        type: 'danger',
                        title: 'Update user failed'
                    })
                    this.$Progress.fail();
                });
            },
            editModal(user){
                this.editmode = true;
                this.form.reset();
                $('#addNewUser').modal('show');
                this.form.fill(user);
            },
            newModal(){
                this.editmode = false;
                this.form.reset();
                $('#addNewUser').modal('show');
            },
            loadUsers(){
                if (this.$gate.isAdminOrAuthor()) {
                    axios.get('api/user').then( ( { data } ) => (this.users = data));
                }
            },
            createUser(){
                this.$Progress.start();
                this.form.post('api/user')
                .then(() => {
                    Fire.$emit('AfterCreate');
                    $('#addNewUser').modal('hide');

                    Toast.fire({
                        type: 'success',
                        title: 'User created successfully'
                    })
                    this.$Progress.finish()
                })
                .catch(() => {

                })
                
            },
            deleteUser(id){
                Swal.fire({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    type: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                    }).then((result) => {

                        if (result.value) {
                            // Send request to 
                            this.form.delete('api/user/' + id)
                            .then(() => {
                                Swal.fire(
                                'Deleted!',
                                'User has been deleted.',
                                'success'
                                )
                                Fire.$emit('AfterCreate');
                            })
                            .catch(() => {
                                swal('Failed!', 'There was something wrong.', 'warning');
                            });
                        }
                        
                    })
            }
        },
        created() {
            Fire.$on('searching', () => {
                let query = this.$parent.search;
                axios.get('api/findUser?q=' + query)
                .then((data) => {
                    this.users = data.data
                    this.usersEmpty();
                })
                .catch(() => {

                })
            })
            this.loadUsers();
            this.usersEmpty();
            Fire.$on('AfterCreate',() => {
                this.loadUsers();
                
            });
            // setInterval(() => this.loadUsers(), 3000);
        }
    }

</script>

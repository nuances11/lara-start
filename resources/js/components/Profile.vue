<style>
    .widget-user .widget-user-header {
        height: 200px;
    }

    .widget-user-header {
        background-position: center center;
        background-size: cover;
    }

    .widget-user .widget-user-image > img {
        width: 120px;
        height: 120px;
        border: 3px solid #ffffff;
    }
</style>
<template>
    <div class="container">
        <div class="row">
            <div class="col-md-12 mt-3">
                <div class="card card-widget widget-user">
                    <!-- Add the bg color to the header using any of the bg-* classes -->
                    <div class="widget-user-header text-white" style="background: url('./img/cover.jpg');">
                        <h3 class="widget-user-username">Elizabeth Pierce</h3>
                        <h5 class="widget-user-desc">Web Designer</h5>
                    </div>
                    <div class="widget-user-image">
                        <img class="img-circle" :src="getProfilePhoto()" alt="User Avatar">
                    </div>
                    <div class="card-footer">
                        <div class="row">
                            <div class="col-sm-4 border-right">
                                <div class="description-block">
                                    <h5 class="description-header">3,200</h5>
                                    <span class="description-text">SALES</span>
                                </div>
                                <!-- /.description-block -->
                            </div>
                            <!-- /.col -->
                            <div class="col-sm-4 border-right">
                                <div class="description-block">
                                    <h5 class="description-header">13,000</h5>
                                    <span class="description-text">FOLLOWERS</span>
                                </div>
                                <!-- /.description-block -->
                            </div>
                            <!-- /.col -->
                            <div class="col-sm-4">
                                <div class="description-block">
                                    <h5 class="description-header">35</h5>
                                    <span class="description-text">PRODUCTS</span>
                                </div>
                                <!-- /.description-block -->
                            </div>
                            <!-- /.col -->
                        </div>
                        <!-- /.row -->
                    </div>
                </div>
            </div>
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header p-2">
                        <ul class="nav nav-pills">
                            <li class="nav-item"><a class="nav-link" href="#activity" data-toggle="tab">Activity</a>
                            </li>
                            <li class="nav-item"><a class="nav-link active show" href="#settings" data-toggle="tab">Settings</a>
                            </li>
                        </ul>
                    </div><!-- /.card-header -->
                    <div class="card-body">
                        <div class="tab-content">
                            <div class="tab-pane" id="activity">

                            </div>
                            <!-- /.tab-pane -->

                            <div class="tab-pane active show" id="settings">
                                <form class="form-horizontal">
                                    <div class="form-group">
                                        <label for="name" class="col-sm-2 control-label">Name</label>

                                        <div class="col-sm-10">
                                            <input v-model="form.name" type="text" class="form-control" id="name" name="name" placeholder="Name" :class="{ 'is-invalid' : form.errors.has('name')}">
                                        </div>
                                        <has-error :form="form" field="name"></has-error>
                                    </div>
                                    <div class="form-group">
                                        <label for="email" class="col-sm-2 control-label">Email</label>

                                        <div class="col-sm-10">
                                            <input v-model="form.email" type="email" class="form-control" id="email" name="email" placeholder="Email" :class="{ 'is-invalid' : form.errors.has('email')}">
                                        </div>
                                        <has-error :form="form" field="enmail"></has-error>
                                    </div>
                                    <div class="form-group">
                                        <label for="Bio" class="col-sm-2 control-label">Bio <span class="disabled">(Optional)</span></label>

                                        <div class="col-sm-10">
                                            <textarea v-model="form.bio" class="form-control" id="inputExperience" placeholder="Say something..." :class="{ 'is-invalid' : form.errors.has('bio')}"></textarea>
                                        </div>
                                        <has-error :form="form" field="bio"></has-error>
                                    </div>
                                    <div class="form-group">
                                        <label for="photo" class="col-sm-2 control-label">Profile Photo</label>

                                        <div class="col-sm-10">
                                            <input type="file" @change="updateProfile" id="photo" name="photo">
                                        </div>
                                    </div>
                                    <div class="form-group">
                                        <label for="password" class="col-sm-12 control-label">Password (Leave empty if not changing)</label>

                                        <div class="col-sm-10">
                                            <input v-model="form.password" type="password" class="form-control" id="password" name="password" :class="{ 'is-invalid' : form.errors.has('password')}">
                                        </div>
                                        <has-error :form="form" field="password"></has-error>
                                    </div>
                                    <div class="form-group">
                                        <div class="col-sm-offset-2 col-sm-10">
                                            <button @click.prevent="updateInfo" type="submit" class="btn btn-success">Update</button>
                                        </div>
                                    </div>
                                </form>
                            </div>
                            <!-- /.tab-pane -->
                        </div>
                        <!-- /.tab-content -->
                    </div><!-- /.card-body -->
                </div>
                <!-- /.nav-tabs-custom -->
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                profilePhoto : '',
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
        mounted() {
            console.log('Profile Component mounted.')
        },

        methods: {
            resetForm() {
                console.log('Form Resetting');
                this.form.photo = null;
                this.form.password = null;
            },
            getUserData() {
                axios.get('api/profile')
                .then(({ data }) => (this.form.fill(data)));
            },
            getProfilePhoto() {
                axios.get('api/profile')
                .then(({ data }) => {
                    this.profilePhoto = data.photo
                });
                return 'img/profile/' + this.profilePhoto;
            },
            updateInfo() {
                this.$Progress.start();
                this.form.put('api/profile/')
                    .then(() => {
                        Fire.$emit('AfterCreate');
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
            updateProfile(e){
                console.log('uploading...');
                let file = e.target.files[0];
                let reader = new FileReader();

                if (file['size'] < 2111775) {
                    reader.onloadend = (file) => {
                        // console.log('RESULT', reader.result)
                        this.form.photo = reader.result;
                    }
                    reader.readAsDataURL(file)
                }else{
                    Toast.fire({
                        type: 'error',
                        title: 'File is too large'
                    })
                }
                
            }
        },

        created() {
            this.getUserData();
            Fire.$on('AfterCreate',() => {
                this.resetForm();
                this.getUserData();
                this.getProfilePhoto();
            });
        }
    }
</script>

<template>
    <div class="container">
        <div class="row mt-5">
          <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Users Table</h3>
                <div class="card-tools">
                    <button class="btn btn-success" @click="newModal()">Add New <i class="fas fa-user-plus fa-fw"></i></button>

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
                    <th>Registered At</th>
                    <th>Modify</th>
                  </tr>

                  <tr v-for="user in users" :key="user.id">
                    <td>{{user.id}}</td>
                    <td>{{user.name}}</td>
                    <td>{{user.email}}</td>
                    <td>{{user.type | upText}}</td>
                    <td>{{user.created_at | myDate}}</td>
                    <td>
                        <a href="#" @click="editModal(user)"> <i class="fas fa-edit blue"></i> </a>
                        /
                        <a href="#" @click="deleteUser(user.id)">
                            <i class="fas fa-trash-alt red"></i>
                        </a>
                    </td>
                  </tr>
                </tbody></table>
              </div>
              <!-- /.card-body -->
            </div>
            <!-- /.card -->
          </div>
        </div>

        <!-- Modal -->
        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="exampleModalCenterTitle" aria-hidden="true">
        <div class="modal-dialog modal-dialog-centered" role="document">
            <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title" v-show="!editmode" id="titleAddNew">Add New</h5>
                <h5 class="modal-title" v-show="editmode" id="titleAddNew">Update User Info</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                <span aria-hidden="true">&times;</span>
                </button>
            </div>

            <form @submit.prevent="editmode ? updateUser() : createUser()">
            <div class="modal-body">

                <div class="form-group">
                <!-- <label>name:</label> -->
                <input v-model="form.name" type="text" name="name"
                    placeholder="Name"
                    class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                <has-error :form="form" field="name"></has-error>
                </div>

                <div class="form-group">
                <!-- <label>name:</label> -->
                <input v-model="form.email" type="email" name="email"
                    placeholder="Email Address"
                    class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                <has-error :form="form" field="email"></has-error>
                </div>


                <div class="form-group">
                <!-- <label>name:</label> -->
                <textarea v-model="form.bio" name="bio" id="bio"
                    placeholder="Short bio for user (Optional)"
                    class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                <has-error :form="form" field="bio"></has-error>
                </div>


                <div class="form-group">
                <!-- <label>name:</label> -->
                <select v-model="form.type" name="type" id="type"
                    class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                    <option value="">Select User Role</option>
                    <option value="admin">Admin</option>
                    <option value="storeKeeper">Store Keeper</option>
                    <option value="procurementCleark">Procurement Cleark</option>
                    <option value="inventoryCleark">Inventory Cleark</option>
                    <option value="deputyBursar">Deputy Bursar</option>
                    <option value="seniorAssistantBursar">Senior Assistant Bursar</option>
                </select>
                <has-error :form="form" field="type"></has-error>
                </div>

                <div class="form-group">
                <!-- <label>name:</label> -->
                <input v-model="form.password" type="password" name="password"
                    placeholder="Password"
                    class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                <has-error :form="form" field="password"></has-error>
                </div>


            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                <button type="submit" v-show="!editmode" class="btn btn-primary">Create</button>
                <button type="submit" v-show="editmode" class="btn btn-success">Update</button>
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
                editmode : false,
                users : {},
                form : new Form({
                    id : '',
                    name : '',
                    email : '',
                    password : '',
                    type : '',
                    bio : '',
                    photo : ''
                })
            }
        },
        methods : {
            updateUser(){
                this.$Progress.start();
                //console.log("Edit mode");
                this.form.put('api/user/'+this.form.id)
                .then(()=>{
                    //success
                    $('#addNew').modal('hide');

                    swal(
                        'Updated!',
                        'User info has been updated.',
                        'success'
                    )
                    this.$Progress.finish();
                    Fire.$emit('afterCreate');
                })
                .catch(()=>{
                    //Fail
                    this.$Progress.fail()

                });
            },
            newModal(){
                this.editmode = false;
                this.form.reset();
                $('#addNew').modal('show');
            },
            editModal(user){
                this.editmode = true;
                this.form.reset();
                $('#addNew').modal('show');
                this.form.fill(user);
            },
            deleteUser(id){

                //Using sweet alert to notify
                swal({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    type: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                    }).then((result) => {

                        if (result.value) {

                        //Sending request to the server
                        this.form.delete('api/user/'+id).then(()=>{

                                swal(
                                'Deleted!',
                                'Your file has been deleted.',
                                'success'
                                )

                                Fire.$emit('afterCreate');
                        }).catch(()=>{
                                swal({
                                type: 'error',
                                title: 'Oops...',
                                text: 'Something went wrong!',
                                })
                        });
                }

            })
        },
            loadUser(){
                axios.get('api/user').then(({data})=>(this.users = data.data));
            },
            createUser(){
                this.$Progress.start();
                this.form.post('api/user')

                .then(()=>{
                    Fire.$emit('afterCreate');

                    $('#addNew').modal('hide');


                    toast({
                        type: 'success',
                        title: 'User Created Successfully'
                    });
                    this.$Progress.finish()
                })
                .catch(()=>{
                    this.$Progress.fail()
                });

            }

        },
        created() {
            //console.log('Component mounted.')
            this.loadUser();
            //setInterval(()=>this.loadUser(),3000);

            Fire.$on('afterCreate',()=>{
                this.loadUser()
                });
        }
    }
</script>

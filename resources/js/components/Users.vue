<template>
    <div class="container">
        <div class="row mt-5">
            <div class="col-md-12">
                <div class="card">
                      <div class="card-header">
                        <h3 class="card-title">List of Users </h3>

                        <div class="card-tools">

                            <button class="btn btn-success" @click="newModal">Add New <i class="fa fa-user-plus fa-fw"></i></button>
                          
                        </div>
                      </div>
                      <!-- /.card-header -->
                      <div class="card-body table-responsive p-0">
                        <table class="table table-hover">
                          <tbody><tr>
                            <th>ID</th>
                            <th>Name</th>
                            <th>Email</th>
                            <th>Created at</th>
                           
                            <th>Modify</th>
                          </tr>
                          <tr v-for="user in users">
                            <td>{{user.id}}</td>
                            <td>{{user.name}}</td>
                            <td>{{user.email}}</td>
                            <td>{{user.created_at | myDate}}</td>
                           
                            <td>
                                <a href="#" @click="editModal(user)">
                                   <i class="fa fa-edit blue"></i>
                                </a>
                                /
                                <a href="#" @click = "deleteUser(user.id)">
                                    <i class="fa fa-trash red"></i>
                                </a>
                            </td>
                          </tr>
                         
                        </tbody></table>
                      </div>
                      <!-- /.card-body -->
                </div>
            </div>
        </div>

        <!-- Modal -->
        <div class="modal fade" id="addNewModal" tabindex="-1" role="dialog" aria-labelledby="addNewModalLabel" aria-hidden="true">
          <div class="modal-dialog modal-dialog-centered" role="document">
            <div class="modal-content">
              <div class="modal-header">
                <h5 class="modal-title" id="addNewModalLabel" v-show="!editMode">Add New</h5>
                <h5 class="modal-title" id="addNewModalLabel" v-show="editMode">Update user</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                  <span aria-hidden="true">&times;</span>
                </button>
              </div>

              <form @submit.prevent="editMode ? updateUser() : createUser()" @keydown="form.onKeydown($event)">
              <!-- Alert -->
              <alert-error :form="form" message="There were errors occured on submitting the form. Please check the errors below and resubmit the form"></alert-error>
              <div class="modal-body">
                   
                        <div class="form-group">
                          <label>Name</label>
                          <input v-model="form.name" type="text" name="name"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                          <has-error :form="form" field="name"></has-error>
                        </div>

                        <div class="form-group">
                          <label>Email</label>
                          <input v-model="form.email" type="text" name="email"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                          <has-error :form="form" field="email"></has-error>
                        </div>

                        <div class="form-group">
                          <label>Password</label>
                          <input v-model="form.password" type="password" name="password"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                          <has-error :form="form" field="password"></has-error>
                        </div>

                        
              </div>
              <div class="modal-footer">
                <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                <button v-show="editMode" type="submit" class="btn btn-success">Update</button>
                <button v-show="!editMode" type="submit" class="btn btn-primary">Create</button>
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
              users: {},  
              form: new Form({
              name: '',
              email: '',
              password: ''
              })
              
            }
        },
        methods : {

           updateUser(){

           },
           newModal(){
             this.editMode = false;
             this.form.reset();
             $('#addNewModal').modal('show');
           },
           editModal(user){
             this.editMode = true;
             this.form.reset();
             $('#addNewModal').modal('show');
             this.form.fill(user);
           },

           loadUsers(){
             
             axios.get('api/user').then(({ data }) => (this.users = data.data));
           } ,

           createUser(){
            this.$Progress.start(); 
            this.form.post('api/user').then(() => {

                  this.$emit('afterCreate'); //fire event

                  //this.createUser = true;

                  Toast.fire({
                    type: 'success',
                    title: 'User created successfully'
                  })

                  $('#addNewModal').modal('hide');

                  this.$Progress.finish();
            })

            .catch(() => {

            })
           

           },

           deleteUser(id) {

             Swal.fire({
              title: 'Are you sure?',
              text: "You won't be able to revert this!",
              type: 'warning',
              showCancelButton: true,
              confirmButtonColor: '#3085d6',
              cancelButtonColor: '#d33',
              confirmButtonText: 'Yes, delete it!'
              }).then((result) => {

               

                if(result.value) {

                   //send request to the server

                  this.form.delete('api/user/'+id).then((response) => {
                 
                  Swal.fire(
                    'Deleted!',
                     response.message,
                    'success'
                  )

                 this.$emit('afterCreate');
                  
                  }).catch(() => {

                     Swal.fire(
                      'Failed!',
                       'There was something wrong',
                      'warning'
                    )

                  });

                } //result.value
              
              })

           }

        },

        

        created() {
            this.loadUsers();
            this.$on('afterCreate', () => {
              this.loadUsers();
            }) //listen to the event

        }
    }
</script>

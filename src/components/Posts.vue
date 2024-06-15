<template>
  <v-sheet class="pa-12" rounded>
    <v-data-table
      :headers="headers"
      :items="postsArray.value"
      :sort-by="[{ key: 'id', order: 'asc' }]"
    >
      <template v-slot:top>
        <v-toolbar
          flat
        >
          <v-toolbar-title>Listado de Posts</v-toolbar-title>
          <v-divider
            class="mx-4"
            inset
            vertical
          ></v-divider>
          <v-spacer></v-spacer>
          
          <v-dialog max-width="500" v-model="formDialog">
            <template v-slot:activator="{ props: activatorProps }">
              <v-btn
                v-bind="activatorProps"
                color="surface-variant"
                text="Nuevo Post"
                variant="flat"
                @click="openformDialog"
              ></v-btn>
            </template>

            <template v-slot:default="{ isActive }">
              <v-card :title="formTitle">
                <v-form ref="form"
                        v-model="isValid"
                        validate-on="input"
                        @submit.prevent="onSubmit"
                      >

                <v-card-text>
                  <v-container>
                      <v-row>
                          <v-col cols="12" md="12" sm="12">
                            <v-text-field
                              v-model="postItem.title"
                              :readonly="loading"
                              :rules="[requiredInput]"
                              label="Título"
                              clearable
                            ></v-text-field>
                          </v-col>
                          <v-col cols="12" md="12" sm="12">
                            <v-text-field
                            v-model="postItem.body"
                            :readonly="loading"
                            :rules="[requiredInput]"
                            label="Texto"
                            placeholder="Ingresa el texto"
                            clearable
                          ></v-text-field>
                          </v-col>

                          <br>
                      </v-row>
                  </v-container>
                </v-card-text>
                <v-card-actions>
                  <v-spacer></v-spacer>
                    <v-btn
                      text="Cerrar"
                      color="blue-darken-1"
                      variant="text"
                      @click="formDialog = false"
                    >
                  </v-btn>

                  <v-btn v-if="!isEditing"
                    :disabled="!isValid"
                    text="Crear"
                    color="blue-darken-1"
                    variant="text"
                    @click="CreatePost"
                  >
                  </v-btn>
                  <v-btn v-else
                    :disabled="!isValid"
                    text="Editar"
                    color="blue-darken-1"
                    variant="text"
                    @click="EditPost"
                  >
                  </v-btn>
                </v-card-actions>
                </v-form>
              </v-card>
            </template>
          </v-dialog>
          <v-dialog v-model="deleteDialog" max-width="500px">
            <v-card>
              <v-card-title class="text-h5">Eliminar: {{postItem.title}}</v-card-title>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue-darken-1" variant="text" @click="deleteDialog = false">Cancel</v-btn>
                <v-btn color="blue-darken-1" variant="text" @click="DeletePost">OK</v-btn>
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template v-slot:item.actions="{ item }">
        <v-icon
          size="small"
          @click="editItemDialog(item)"
        >
          mdi-pencil
        </v-icon>
        <v-icon
          size="small"
          @click="deleteItemDialog(item)"
        >
          mdi-delete
        </v-icon>
      </template>
      <template v-slot:no-data>
        <v-btn
          color="primary"
          @click="GetAllPost()"
        >
          Cargar Post
        </v-btn>
      </template>
    </v-data-table>
  </v-sheet>
</template>

<script>
    import { ref, onMounted } from 'vue'
    import axios from 'axios'
    import { toast } from 'vue3-toastify'
    import 'vue3-toastify/dist/index.css'

    export default {
    data () {
        return {
            // Form Variables
            form: ref(null),
            loading: ref(false),
            isEditing: ref(false),
            isValid: ref(null),
            formTitle: ref(''),
            // Post Variables
            postsArray: ref([]),
            postId: ref(0),
            postItem: ref({
              title: '',
              body: '',
            }),
            // Dialog Variables
            formDialog: ref(false),
            deleteDialog: ref(false),
            // Data Table Variables
            headers: ref([
              {
                align: 'start',
                key: 'id',
                sortable: true,
                title: 'ID',
              },
              { key: 'title', title: 'Título' },
              { key: 'body', title: 'Texto' },
              { key: 'actions', title: 'Acciones', sortable: false },
            ]),
        }
    },
    async mounted(){
      this.GetAllPost()
    },
    methods: {
      // CRUD Methods
      async CreatePost () {
        const { valid } = await this.$refs.form.validate()
        if (!valid) return

        this.CreatePostAPI(this.postItem)
          .then(async response =>{
            toast.success('¡Se ha creado exitosamente!', { autoclose: 1000 } )
            this.$refs.form.reset()
            this.GetPost(response.data.id)
          })
          .catch(error => {
            console.log(error)
            toast.error('Fallo: ' + error)
          })
      },
      async EditPost () {
        const { valid } = await this.$refs.form.validate()
        if (!valid) return

        this.EditPostAPI(this.postId, this.postItem)
          .then(async response =>{
            toast.success('¡Se ha editado exitosamente!', { autoclose: 1000 } )
            this.$refs.form.reset()
            this.GetPost(response.data.id)
          })
          .catch(error => {
            console.log(error)
            toast.error('Fallo: ' + error)
          })
      },
      async DeletePost () {
        this.DeletePostAPI(this.postId)
          .then(async response =>{
            let index = this.postsArray.value.findIndex(item => item.id === this.postId)
            this.postsArray.value.splice(index, 1)
            this.deleteDialog = false
            toast.success('¡Se ha eliminado exitosamente!', { autoclose: 1000 } )
          })
          .catch(error => {
            console.log(error)
            toast.error('Fallo: ' + error)
          })
      },
      async GetAllPost (){
        this.GetAllPostAPI()
          .then(response => {
            this.postsArray.value = response.data
          })
          .catch(error => {
            console.log(error)
          })
      },
      async GetPost(id) {
        this.GetPostAPI(id)
          .then(async response => {
            let index = this.postsArray.value.findIndex(item => item.id === response.data.id)
            if (index === -1){ // If it doesn't exist add to Array (Used on Create)
              this.postsArray.value.push(response.data)
            }else { // Else Replace value on the same Index (Used on Edit)
              this.postsArray.value.splice(index, 1, response.data)
            }
          })
          .catch(error => {
            console.log(error)
            toast.error('Fallo: ' + error)
        })
      },
      // REST Methods API 
      async CreatePostAPI (post) { return axios.post("api/posts", post) },
      async GetPostAPI (id) { return axios.get("api/posts/"+id) },
      async GetAllPostAPI () { return axios.get('/api/posts') },
      async EditPostAPI (id, post) { return axios.put("api/posts/"+id, post) },
      async DeletePostAPI (id) { return axios.delete("api/posts/"+id)  },
      // Dialog Methods
      openformDialog () {
        this.postId = 0
        this.postItem = {}
        this.formDialog = true
        this.formTitle = "Crear Nuevo Post"
        this.isEditing = false
      },
      editItemDialog (item) {
        this.postId = item.id
        this.postItem = { 
          title: item.title, 
          body: item.body,
        }
        this.formDialog = true
        this.formTitle = "Editar Post"
        this.isEditing = true
      },
      deleteItemDialog (item) {
        this.postId = item.id
        this.postItem = { 
          title: item.title, 
          body: item.body,
        }
        this.deleteDialog = true
      },
      onSubmit () {
        const { valid } = this.$refs.form.validate()
        if (!valid) return
      },
      requiredInput (input) {
        return !!input || 'Campo requerido'
      },
    }
  }
</script>

<style scoped>
</style>

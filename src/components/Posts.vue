

<template>
  <v-sheet class="pa-12" rounded>
    <v-data-table
      :headers="headers"
      :items="postsArray"
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

<script setup>
  import { ref, onMounted } from 'vue'
  import axios from 'axios'
  import { toast } from 'vue3-toastify'
  import 'vue3-toastify/dist/index.css'

  // Post Variables
  const postsArray = ref([])
  const postId = ref(0)
  const postItem = ref({
    title: '',
    body: '',
  })
  // Form Variables
  const form = ref(null) // Para acceder al ref Form lo inicializamos como Null
  const loading = ref(false)
  const isEditing = ref(false)
  const isValid = ref(null)
  const formTitle = ref('')
  // Dialog Variables
  const formDialog = ref(false)
  const deleteDialog = ref(false)
  // Data Table Variables
  const headers = ref([
    {
      align: 'start',
      key: 'id',
      sortable: true,
      title: 'ID',
    },
    { key: 'title', title: 'Título' },
    { key: 'body', title: 'Texto' },
    { key: 'actions', title: 'Acciones', sortable: false },
  ])

  // Methods:
  onMounted(async() => {
    GetAllPost()
  })

  const onSubmit = () => {
      const { valid } = form.value.validate()
      if (!valid) return
      loading = true
      setTimeout(() => (loading = false), 2000)
  }

  // Post Methods:
  const CreatePost = async() => {
    const { valid } = await form.value.validate()
    if (!valid) return

    CreatePostAPI(postItem.value)
      .then(async response =>{
        toast.success('¡Se ha creado exitosamente!', { autoclose: 1000 } )
        form.value.reset()
        GetPost(response.data.id)
      })
      .catch(error => {
        console.log(error)
        toast.error('Fallo: ' + error)
      })
  }

  const EditPost = async() => {
    const { valid } = await form.value.validate()
    if (!valid) return

    EditPostAPI(postId.value, postItem.value)
      .then(async response =>{
        toast.success('¡Se ha editado exitosamente!', { autoclose: 1000 } )
        form.value.reset()
        GetPost(response.data.id)
      })
      .catch(error => {
        console.log(error)
        toast.error('Fallo: ' + error)
      })
  }

  const DeletePost = async() => {
    DeletePostAPI(postId.value)
      .then(async response =>{
        let index = postsArray.value.findIndex(item => item.id === postId.value)
        postsArray.value.splice(index, 1)
        deleteDialog.value = false
        toast.success('¡Se ha eliminado exitosamente!', { autoclose: 1000 } )
      })
      .catch(error => {
        console.log(error)
        toast.error('Fallo: ' + error)
      })
  }

  const GetAllPost = async() => {
    GetAllPostAPI()
      .then(response => {
        postsArray.value = response.data
      })
      .catch(error => {
        console.log(error)
        toast.error('Fallo: ' + error)
      })
  }

  const GetPost = async(id) => {
    GetPostAPI(id)
      .then(async response => {
        let index = postsArray.value.findIndex(item => item.id === response.data.id)
        if (index === -1){ // If it doesn't exist add to Array (Used on Create)
          postsArray.value.push(response.data)
        }else { // Else Replace value on the same Index (Used on Edit)
          postsArray.value.splice(index, 1, response.data)
        }
      })
      .catch(error => {
        console.log(error)
        toast.error('Fallo: ' + error)
    })
  }

  // REST Methods API
  const CreatePostAPI = async(post) => { return axios.post("api/posts", post) }
  const GetPostAPI = async(id) => { return axios.get("api/posts/"+id) }
  const GetAllPostAPI = async() => { return axios.get('/api/posts') }
  const EditPostAPI = async(id, post) => { return axios.put("api/posts/"+id, post) }
  const DeletePostAPI = async(id) => { return axios.delete("api/posts/"+id) }

  // Dialog Methods
  const openformDialog = () => {
    postId.value = 0
    postItem.value = {}
    formDialog.value = true
    formTitle.value = "Crear Nuevo Post"
    isEditing.value = false
  } 

  const editItemDialog = (item) => {
    postId.value = item.id
    postItem.value = Object.assign({}, item)
    formDialog.value = true
    formTitle.value = "Editar Post"
    isEditing.value = true
  }

  const deleteItemDialog = (item) => {
    postId.value = item.id
    postItem.value = Object.assign({}, item)
    deleteDialog.value = true
  }

  // Validation Methods:
  const requiredInput = (input) => {
    return !!input || 'Campo requerido'
  }
</script>

<style scoped>
</style>

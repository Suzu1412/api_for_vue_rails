

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
        <v-toolbar-title>My CRUD</v-toolbar-title>
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
                      validate-on="input lazy"
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

                <v-btn
                  :disabled="!isValid"
                  text="Crear"
                  color="blue-darken-1"
                  variant="text"
                  @click="CreatePost"
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
        @click="initialize"
      >
        Reset
      </v-btn>
    </template>
  </v-data-table>


    <v-data-table
      :headers="headers"
      :items="postsArray"
      :search="search"
      item-key="id"
      items-per-page="5"
    ></v-data-table>
  </v-sheet>


  </template>

<script setup>
  import { ref, onMounted } from 'vue'
  import axios from 'axios'
  import { toast } from 'vue3-toastify'
  import 'vue3-toastify/dist/index.css'

  const postsArray = ref([])
  const postId = ref(0)
  const form = ref(null) // Para acceder al ref Form lo inicializamos como Null
  const loading = ref(false)
  const isEditing = ref(false)
  const isValid = ref(null)
  const formTitle = ref('')
  const formDialog = ref(false)
  const deleteDialog = ref(false)

  // Data Table Variables
  // const dialog = ref('')
  const search = ''
  const headers = ref([
          {
            align: 'start',
            key: 'id',
            sortable: true,
            title: 'ID',
          },
          { key: 'title', title: 'Título' },
          { key: 'body', title: 'Texto' },
          { title: 'Actions', key: 'actions', sortable: false },
        ])

  const postItem = ref({
    title: '',
    body: '',
  })


  onMounted(async() => {
    axios.get('/api/posts')
      .then(response => {
        postsArray.value = response.data
      })
      .catch(error => {
        console.log(error)
        toast.error('Fallo: ' + error)
      })
  })

  // Methods:
  const onSubmit = () => {
    const { valid } = this.$refs.form.validate()

        if (!valid) return

        loading = true

        setTimeout(() => (loading = false), 2000)
  }

  const CreatePost = async() => {
    axios.post("api/posts", postItem.value)
        .then(async response =>{
          toast.success('¡Se ha creado exitosamente!', { autoclose: 1000 } )
          form.value.reset()
          const newPost = await axios.get("api/posts/"+response.data.id)
          postsArray.value.push(newPost.data)
        })
        .catch(error => {
          console.log(error)
          toast.error('Fallo: ' + error)
        })
  }

  const GetPost = async(id) => {
    axios.get("api/posts/"+id)
    .then(response => {
      return response.data
    })
    .catch(error => {
      console.log(error)
      toast.error('Fallo' + error)
      return null
    })
  }

  const DeletePost = async() => {
    axios.delete("api/posts/"+postId.value)
        .then(async response =>{
          postsArray.value.splice(postsArray.value.findIndex(item => item.id === postId.value), 1)
          deleteDialog.value = false
          toast.success('¡Se ha eliminado exitosamente!', { autoclose: 1000 } )
        })
        .catch(error => {
          console.log(error)
          toast.error('Fallo: ' + error)
        })
  }

  // Dialog Methods
  const openformDialog = () => {
    postId.value = 0
    postItem.value = {}
    formDialog.value = true
    formTitle.value = "Crear Nuevo Post"
  } 

  const editItemDialog = (item) => {
    postId.value = postsArray.value.indexOf(item)
    postItem.value = Object.assign({}, item)
    formDialog.value = true
    formTitle.value = "Editar Post"
  }

  const deleteItemDialog = (item) => {
    postId.value = item.id
    postItem.value = Object.assign({}, item)
    deleteDialog.value = true
  }

  // Validation Methods:
  const resetInput = () => {
    form.value.reset()
  }

  const requiredInput = (input) => {
    return !!input || 'Campo requerido'
  }
</script>


<style scoped>
</style>

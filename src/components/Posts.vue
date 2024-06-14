

<template>
<v-sheet class="pa-12" rounded>
    <v-card class="mx-auto px-6 py-8" max-width="344">
      <v-form ref="form"
        v-model="isValid"
        validate-on="input lazy"
        @submit.prevent="onSubmit"
      >
        <v-text-field
          v-model="title"
          :readonly="loading"
          :rules="[requiredInput]"
          class="mb-2"
          label="Título"
          clearable
        ></v-text-field>

        <v-text-field
          v-model="body"
          :readonly="loading"
          :rules="[requiredInput]"
          label="Texto"
          placeholder="Ingresa el texto"
          clearable
        ></v-text-field>

        <br>

        <v-btn v-if="isEditing"
          @click="updatePost"
          :disabled="!isValid"
          :loading="loading"
          color="success"
          size="large"
          type="submit"
          variant="elevated"
          block
        >
          Actualizar Post
        </v-btn>

        <v-btn v-if="isEditing"
          @click="cancelEdit"
          :loading="loading"
          color="error"
          size="large"
          type="submit"
          variant="elevated"
          block
        >
          Cancelar
        </v-btn>

        <v-btn v-else
          @click="SubmitCreatePost"
          :disabled="!isValid"
          :loading="loading"
          color="primary"
          size="large"
          type="submit"
          variant="elevated"
          block
        >
          Crear nuevo Post
        </v-btn>

        <v-btn
          @click="resetInput"
          color="info"
          size="large"
          type="submit"
          variant="elevated"
          block
        >
          Reset
        </v-btn>

      </v-form>
    </v-card>


    <v-dialog max-width="500">
  <template v-slot:activator="{ props: activatorProps }">
    <v-btn
      v-bind="activatorProps"
      color="surface-variant"
      text="Open Dialog"
      variant="flat"
    ></v-btn>
  </template>

  <template v-slot:default="{ isActive }">
    <v-card title="Dialog">
      <v-card-text>
        Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
      </v-card-text>

      <v-card-actions>
        <v-spacer></v-spacer>

        <v-btn
          text="Close Dialog"
          @click="isActive.value = false"
        ></v-btn>
      </v-card-actions>
    </v-card>
  </template>
</v-dialog>




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
        
        <v-dialog max-width="500">
          <template v-slot:activator="{ props: activatorProps }">
            <v-btn
              v-bind="activatorProps"
              color="surface-variant"
              text="Nuevo Post"
              variant="flat"
            ></v-btn>
          </template>

          <template v-slot:default="{ isActive }">
            <v-card :title="formTitle">
              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col
                      cols="12"
                      md="12"
                      sm="12"
                    >
                      <v-text-field
                        v-model="postItem.title"
                        label="Título"
                      ></v-text-field>
                    </v-col>
                    <v-col
                      cols="12"
                      md="12"
                      sm="12"
                    >
                      <v-text-field
                        v-model="postItem.body"
                        label="Texto"
                      ></v-text-field>
                    </v-col>
                    
                    
                    
                  </v-row>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>

                <v-btn
                  text="Close Dialog"
                  @click="isActive.value = false"
                ></v-btn>
              </v-card-actions>
            </v-card>
          </template>
        </v-dialog>


        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="text-h5">Are you sure you want to delete this item?</v-card-title>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue-darken-1" variant="text" @click="closeDelete">Cancel</v-btn>
              <v-btn color="blue-darken-1" variant="text" @click="deleteItemConfirm">OK</v-btn>
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:item.actions="{ item }">
      <v-icon
        class="me-2"
        color="info"
        size="small"
        @click="editItem(item)"
      >
        mdi-pencil
      </v-icon>
      <v-icon
        size="small"
        @click="deleteItem(item)"
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
  const title = ref('')
  const body = ref('')
  const form = ref(null) // Para acceder al ref Form lo inicializamos como Null
  const loading = ref(false)
  const isEditing = ref(false)
  const isValid = ref(null)
  const formTitle = 'Crear nuevo formulario'

  // Data Table Variables
  // const dialog = ref('')
  const isa = ref(false)
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

  const SubmitCreatePost = async() => {
    console.log(title)
    const post = {
          title: title.value,
          body: body.value,
    }

    axios.post("api/posts", post)
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

  const resetInput = () => {
    form.value.reset()
  }

  const requiredInput = (input) => {
    return !!input || 'Campo requerido'
  }
</script>


<style scoped>
</style>

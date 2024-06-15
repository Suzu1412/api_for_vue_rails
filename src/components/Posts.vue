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
            title: null,
            body: null,
            isValid: false,
            loading: false,
            isEditing: false,
            posts: [],
            postsArray: ref([]),
        }
    },
    async mounted(){
      this.GetAllPost()
    },
    methods: {
      async createPost () {
        const { valid } = await this.$refs.form.validate()
        if (!valid) return

        const post = {
          title: this.title,
          body: this.body,
        }
        axios.post("api/posts", post)
        .then(response =>{
          axios.get('/api/posts/' + response.data.id)
          .then(response => {
            console.log(response.data)
          })
          .catch(error => {
            console.log(error)
          })

          this.$refs.form.reset()
          toast.success('¡Se ha creado exitosamente!', { autoclose: 1000 } )
        })
        .catch(error => {
          console.log(error)
          toast.error('Fallo' + error)
        })

        // posts.value.push(dataResponse)
        
      },
      async updatePost () {
        console.log("Actualizando post");
      },
      reset () {
        this.$refs.form.reset()
      },
      // Post Methods:
      async GetAllPost (){
        axios.get('/api/posts')
          .then(response => {
            console.log(this.postsArray)
            this.postsArray.value = response.data
          })
          .catch(error => {
            console.log(error)
          })
      },
      onSubmit () {
        const { valid } = this.$refs.form.validate()
        if (!valid) return
      },
    required (input) {
        return !!input || 'Campo requerido'
      },
    }
  }
</script>

<style scoped>
</style>

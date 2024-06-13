

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
          :rules="[required]"
          class="mb-2"
          label="Título"
          clearable
        ></v-text-field>

        <v-text-field
          v-model="body"
          :readonly="loading"
          :rules="[required]"
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
          @click="reset"
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

    <li v-for="(post, index) in postsArray" :key="index" class="list-group-item">{{index + 1}}. {{post}}</li>

  </v-sheet>


  </template>

<script setup>
  import { ref, onMounted } from 'vue'
  import axios from 'axios'
  import { VForm } from 'vuetify/components';

  const postsArray = ref([])
  const title = ref('')
  const body = ref('')
  const form = ref(null) // Para acceder al ref Form lo inicializamos como Null

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
  const SubmitCreatePost = async() => {
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
</script>

<script>
    import { ref, onMounted } from 'vue'
    import axios from 'axios'
    import { toast } from 'vue3-toastify'
    import 'vue3-toastify/dist/index.css'

    const API_URL = "http://localhost:3000/posts"   

    const updatePost = async() => {
        return true;
    }

    const cancelEdit = () => {
        return true;
    }

    const newPosts = ref([])
    const info = ref([])

    export default {
    data () {
        return {
            title: null,
            body: null,
            isValid: false,
            loading: false,
            isEditing: false,
            posts: []
        }
    },
    async mounted(){
      //console.log(newPosts)
      // const { valid } = await this.$refs.form.validate()

      //const res = await axios.get(API_URL)
      // newPosts.value = res.data

      //console.log(res.data)

      //console.log(newPosts.value[0])

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
      onSubmit () {

        //const { valid } = this.$refs.form.validate()

        //if (!valid) return

        //this.loading = true

        //setTimeout(() => (this.loading = false), 2000)
      },
    required (input) {
        return !!input || 'Campo requerido'
      },
    }
  }
</script>

<style scoped>
</style>

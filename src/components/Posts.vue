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
          @click="createPost"
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
  </v-sheet>

  </template>
<script>
    import { ref, onMounted } from 'vue'
    import { toast } from 'vue3-toastify'
    import 'vue3-toastify/dist/index.css'

    const API_URL = "http://localhost:3000/posts"   

    const updatePost = async() => {
        return true;
    }

    const cancelEdit = () => {
        return true;
    }

    const posts = ref([])

    export default {
    data () {
        return {
            title: null,
            body: null,
            isValid: false,
            loading: false,
            isEditing: false,
        }
    },
    async mounted(){
      const { valid } = await this.$refs.form.validate()
      console.log(valid);
    },
    methods: {
      async createPost () {
        const { valid } = await this.$refs.form.validate()
        if (!valid) return

        const res = await fetch(API_URL, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                "Access-Control-Allow-Origin": "*"
            },
            body: JSON.stringify({
                title: this.title,
                body: this.body
            })
        } )

        const dataResponse = await res.json()
        posts.value.push(dataResponse)
        this.$refs.form.reset()
        toast.success('Wow so easy !', { autoclose: 1000 } )
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

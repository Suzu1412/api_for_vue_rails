<template>
<v-sheet class="pa-12" rounded>
    <v-card class="mx-auto px-6 py-8" max-width="344">
      <v-form ref ="form"
        validate-on="input"
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
          :disabled="!form"
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
          :disabled="!form"
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
          :disabled="!valid"
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

    const posts = ref([])
    const API_URL = "http://localhost:3000/posts"   

    const updatePost = async() => {
        return true;
    }

    const cancelEdit = () => {
        return true;
    }

    const createPost = async() => {
        const res = await fetch(API_URL, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                title: title.value,
                body: body.value
            })
        } )

        const dataResponse = await res.json()

        posts.value.push(data);
        title: null
    }

    export default {
    data: () => ({
      form: false,
      title: null,
      body: null,
      valid: false,
      loading: false,
      isEditing: false,
    }),

    methods: {
      async createPost () {
        const { valid } = await this.$refs.form.validate()
        if (!valid) return

        console.log("Creando post");
      },
      async updatePost () {
        console.log("Actualizando post");
      },
      reset () {
        this.$refs.form.reset()
      },
      onInput(){
        console.log("escribiendo");
        valid = this.$refs.form.validate()
      },


      onSubmit () {

        console.log("escribiendo");
        valid = this.$refs.form.validate()
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

<template>
  <div class="course-page">
    <header class="m-3">
      <button class="btn btn-dark" @click.prevent="this.$router.push('/courses')">VOLTAR</button>
    </header>

    <div class="create d-flex">
      <h1 style="text-align:center;" class="titles m-3">ATUALIZAR CURSO</h1>
      <h3 class="titles">Título</h3>
      <div class="form-floating mb-3 w-100">
        <input type="text" class="form-control" id="floatingInput" v-model="title" placeholder="Title">
        <label for="floatingInput">Título do curso:</label>
      </div>

      <h3 class="titles">Descrição</h3>
      <div class="form-floating mb-3 w-100">
        <input type="text" class="form-control" id="description" v-model="description" placeholder="Description">
        <label for="description">Descrição do curso:</label>
      </div>

      <h3 class="titles">Duração</h3>
      <div class="form-floating mb-3 w-100">
        <input type="text" id="time" v-mask="['##:##', '###:##']" class="form-control" v-model="time" placeholder="HH:mm">
        <label for="time">HH:mm</label>
      </div>

      <h3 class="titles">Categoria</h3>
      <select class="w-100" v-model="categoryId">
        <option v-for="category in categories" :value="category.id" :key="category.id">
          {{ category.name }}
        </option>
      </select>
      <h3 class="titles">URL <small style="font-size:12px"> <img :src="info"> vídeo previamente upado no YouTube.</small></h3>
      <div class="form-floating mb-3 w-100">
        <input type="text" id="URL" class="form-control" v-model.trim="url" placeholder="URL do vídeo">
        <label for="URL">URL do vídeo:</label>
      </div>

      <div class="d-flex w-100">
        <button style="background:#F50057; color: white" @click="this.$router.push('/courses')" class="btn w-50">CANCELAR</button>
        <button style="background:#00BFA6; color: white" class="btn w-50" @click.prevent="updateCourse()">ATUALIZAR</button>
      </div>
    </div>
    <img class="background" :src="course" alt="cadastroCurso">
    <div class="loading" v-show="loading">
      <Spinner width="100px" height="100px"/> 
    </div>
  </div>
</template>

<script>
import { defineComponent } from 'vue';

import Spinner from '../../components/Spinner.vue';
import api from '../../api.js';

export default defineComponent( {
  components: {
    Spinner
  },
  
  data() {

    return {
      course: require('@/assets/createCourse.svg'),
      info: require('@/assets/icons/info.svg'),

      time: null,
      title: null,
      description: null,
      url: null,
      categoryId: null,

      courses: [],
      categories: [],
      showEdit: null,
      loading: false,
      error: false,
    }
  },
  methods: {
    async getCategories() {
        const categories = await api.get('api/categories')
        this.categories = categories.data.data.data
    },
    async getCourse() {
        const course = await api.get(`api/courses/${this.$route.query.id}`)
        this.time = course.data.data.time
        this.title = course.data.data.name
        this.url = course.data.data.url
        this.description = course.data.data.detail
        this.categoryId = course.data.data.categories[0].id
    },

    async updateCourse() {

      try {
        this.loading = true;
        let cursePayload = {
          name: this.title,
          detail: this.description,
          categoryId: this.categoryId,
          url: this.url,
          time: this.time,
        };

        if (!cursePayload.name || !cursePayload.detail || !cursePayload.time || !cursePayload.categoryId || !cursePayload.url) {

          if (!cursePayload.name) this.error = true;
          if (!cursePayload.detail) this.error = true;
          if (!cursePayload.categoryId) this.error = true;

          this.$toast.open({ message: 'Preencha todos os campos', type: 'error' })
          this.loading = false;
          return;
        }
  
        await api.put(`api/courses/${this.$route.query.id}`, cursePayload).then(() => {
          this.$toast.open({ message: 'curso atualizado com sucesso!', type: 'success' })
          this.$router.push('/courses')
        });
        
      } catch (error) {
        this.$toast.open({ message: 'erro ao atualizar curso', type: 'error' })
        this.loading = false;
        console.error(error);
      }
    },

    formatUrl(url) {

      let formattedUrl = (url.match(/v=(.*$)/) || [])[1] || url;
      formattedUrl = formattedUrl.split("&")[0];

      return formattedUrl;
    }

  },
  mounted() {
    this.getCategories();
    this.getCourse();
  }

})
</script>

<style lang="scss" scoped>
  button { height: 50px }
  #courses {
    display: flex;
    flex-flow: wrap;
    align-items: center;
    justify-content: center;
    max-width: 100%;
    overflow: auto;
    flex-flow: row wrap;
    position: relative;
    height: 100%;
  }
  .m--5 {
    margin: 5px;
  }
  .course-page {
    padding: 0 0px;
    min-height: 100vh;
    display: flex;
    justify-content: flex-start;
    align-items: center;
    flex-direction: column;

  }
  .create {
    max-width: 800px;
    width: 100%;
    background: #051d3b;
    min-height: 100vh;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    border-top-left-radius: 8px;
    border-top-right-radius: 8px;
    border: 2px solid #6C63FF;
    padding: 20px;
  }
  input, select {
    background: rgb(255, 255, 255);
    color: #151515;
    display: block;
    height: 57px;
    font-size: 18px;
    outline: 0px;
    border-width: 4px 0px;
    border-right-style: initial;
    border-left-style: initial;
    border-right-color: initial;
    border-left-color: initial;
    border-image: initial;
    border-top-style: solid;
    border-top-color: transparent;
    border-bottom-style: solid;
    border-bottom-color: #6C63FF;
    padding: 12px;
    margin-bottom: 45px;
    resize: none;
    border-radius: 4px;
    transition: border-color 0.3s ease 0s;
  }
  .titles {
    width: 100%;
    text-align: left;
    color: white;
  }
  .background {
    position: fixed;
    z-index: -1;
    width: 100%;
    left: 0;
    top: 0;
  }
  .loading {
    position: absolute;
    display: flex;
    justify-content: center;
    align-items: center;
    bottom: 0;
    left: 0;
    top: 0;
    right: 0;
    background: #15151567;
  }
</style>
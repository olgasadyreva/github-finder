<template>
  <div class="wrapper-content wrapper-content--fixed">
    <section>
      <div class="container">

        <!-- preloader -->
        <preloader v-if="loading" :width="70" :height="70"/>

        <!-- erorrs -->
        <div class="error" v-if="error" style="margin-bottom: 20px;">
          <p> {{ error }} </p>
        </div>

        <!-- search -->
        <search
          v-if="!loading"
          :value="search"
          placeholder="Type username..."
          @search="search = $event"/>

        <!-- buttons -->
        <button v-if="!repos" class="btn btnPrimary" @click="getRepos">Search!</button>
        <button v-else class="btn btnPrimary" @click="getRepos">Search Again!</button>

        <!-- info -->
        <div class="info" v-if="repos">
          <div class="info__left">
            <img width="200px" v-bind:src="info.avatar_url" alt="Аватар" />
          </div>
          <div class="info__right">
            <h3>{{info.name}}</h3>
            <span>Репозиториев: {{info.public_repos}}</span>
          </div>
        </div>

        <!-- wrapper -->
        <div class="repos__wrapper" v-if="repos">
          <div class="repos-header">
              <p @click="sort('name')" >Name &#8595;</p>
              <p @click="sort('stargazers_count')">Stars &#8595;</p>
            </div>
          <!-- item -->
          <div class="repos-item" v-for="repo in reposSort" :key="repo.id">
            <div class="repos-info">
              <a class="link" target="_blank" :href="repo.html_url">{{ repo.name }}</a>
              <span > {{ repo.stargazers_count }} ⭐</span>
            </div>
          </div>
        </div>
      </div>
    </section>
    <!-- pagination -->
    <section v-if="repos">
      <div class="container">
        <div class="button-list">
          <div class="btn btnPrimary" @click="prevPage"> &#8592; </div>&nbsp;&nbsp;
          <div class="btn btnPrimary" @click="nextPage"> &#8594; </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import search from '@/components/UI/Search.vue'
import axios from 'axios'

// UI
import preloader from '@/components/UI/Preloader.vue'

export default {
  components: { search, preloader },
  data () {
    return {
      search: '',
      error: null,
      repos: null,
      info: null,
      currentSort: 'name',
      currentSortDir: 'asc',
      page: {
        current: 1,
        length: 3
      },
      loading: false
    }
  },
  computed: {
    reposSort() {
      return this.repos.sort((a, b) => {
        let mod = 1
        if (this.currentSortDir === 'desc') mod = -1
        if (a[this.currentSort] < b[this.currentSort]) return -1 * mod
        if (a[this.currentSort] > b[this.currentSort]) return 1 * mod
        return 0
      }).filter((row, index) => {
        let start = (this.page.current-1)*this.page.length
        let end = this.page.current * this.page.length
        if (index >= start && index < end) return true
      })
    }
  },
  methods: {
    getRepos () {
      this.loading = true
      axios
      .get(`https://api.github.com/users/${this.search}`)
          .then(res => {
            this.error = null
            this.info = res.data
            this.loading = true
            axios
              .get(`https://api.github.com/users/${this.search}/repos`)
              .then(res => {
                this.error = null
                this.repos = res.data
           })
          .catch(err => {
            console.log(err)
            this.repos = null
            this.error = 'Can`t find this user'
            })
            .finally(() => (this.loading = false))
          })
    },
    sort(e) {
      if ( e === this.currentSort) {
        this.currentSortDir = this.currentSortDir === 'asc' ? 'desc' : 'asc'
      }
      this.currentSort = e
    },
    // Pagination
    prevPage () {
      if (this.page.current > 1) this.page.current-=1
    },
    nextPage () {
      if ((this.page.current * this.page.length) < this.repos.length) this.page.current+=1
    }
  }
}
</script>

<style lang="scss" scoped>
.container {
  display: flex;
  align-items: center;
  flex-direction: column;
}
button {
  margin-top: 40px;
}
.repos__wrapper {
  width: 400px;
  margin: 30px 0;
}
.repos-header {
  display: flex;
  justify-content: space-between;
  padding: 10px 0;
  border-bottom: 1px solid #dbdbdb;
  border-top: 1px solid #dbdbdb;
  & p {
    cursor: pointer;
    font-weight: bold
  }
}
.repos-info {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 10px;
  padding: 10px 0;
  border-bottom: 1px solid #dbdbdb;
}
.info {
  display: flex;
  margin: 30px auto;
  border: 1px solid #dbdbdb;
}
.info__right {
  display: flex;
  justify-content: space-around;
  flex-direction: column;
  align-items: center;
  width: 200px;
  & h3 {
    font-weight: bold;
    font-size: 20px;
  }
}
</style>

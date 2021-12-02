<template>
  <div>
    <v-container>
      <p>
        Encontre informações sobre todos os países do mundo
      </p>
      <v-row>
        <v-select 
          :items="filters" 
          item-text="nome" 
          item-value="valor"
          label="Filtrar por"
          v-model="firstFilter"
          class="col-3"
        />

        <v-select 
          v-if="firstFilter"
          :items="secondFilter" 
          item-text="nome" 
          item-value="valor" 
          label="Escolha uma opção"
          v-model="secondFilterSelected"
          class="col-3"
        />

        <v-btn 
          color="purple darken-1 white--text"
          @click="searchCountry"
        >
          Pesquisar
        </v-btn>
      </v-row>
    </v-container>
    <v-container>
      <v-row>
        <p v-if="firstFilter !== null & secondFilterSelected !== null && search === true && countriesList.length > 1">
          Países Selecionados
        </p>
        <p v-else-if="firstFilter !== null & secondFilterSelected !== null && search === true && countriesList.length === 1">
          País Selecionado
        </p>
        <p v-else>
          Todos os Países - Página {{page}}
        </p>
      </v-row>
      <v-container>
        <v-row>
          <v-col v-for="country in countriesList" :key="country.name">
            <nuxt-link :to="country.name">
              <v-img
                :src="country.flag"
                max-height="150"
                max-width="250"
                class="col-6"
              />
            </nuxt-link>
          </v-col>
        </v-row>
      </v-container>
    </v-container>
    <v-container v-if="totalPages > 1">
      <v-row justify="center">
        <v-col cols="8">
          <v-container class="max-width">
            <v-pagination
              v-model="page"
              class="my-4"
              :length="totalPages"
              @input="next"
            ></v-pagination>
          </v-container>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>
<script>

export default {
  data() {
    return {
      page: 1,
      pageSize: 12,
      totalPages: null,
      firstFilter: null,
      secondFilterSelected: null,
      search: false,
      filters: [
        {nome: 'Capital', valor: 'capital'},
        {nome: 'Código de Ligação', valor: 'callingcode'},
        {nome: 'Idioma', valor: 'lang'},
        {nome: 'País', valor: 'name'},
        {nome: 'Região', valor: 'region'},
      ],
      continents: [
        {nome: 'África', valor: 'africa'},
        {nome: 'América', valor: 'americas'},
        {nome: 'Ásia', valor: 'asia'},
        {nome: 'Europa', valor: 'europe'},
        {nome: 'Oceania', valor: 'oceania'}
      ],
      countries: [],
      countriesList: []
    }
  },
  methods: {
    async getCountries(){
      const countries = await this.$axios.$get('/all')
      this.countries = countries
      this.totalPages = parseInt(countries.length / this.pageSize)
      this.countriesList = countries.slice(0,12)
    },
    async searchCountry(){
      var getCountry = await this.$axios.$get(`/${this.firstFilter}/${this.secondFilterSelected}`)
      this.countriesList = getCountry
      this.search = true
    },
    next(){
      if (this.page === 1) {
        this.countriesList = this.countries.slice(0, this.page*12)
      }else{
        this.countriesList = this.countries.slice(this.page*12, this.page*12+12)
      }
    }
  },
  computed: {
    capitals(){
      var capitals = this.countries.map(country => {

        return {
          nome: country.capital,
          valor: country.capital
        }
      })

      return capitals

    },
    language(){
      var language = this.countries.map(country => {
        var languageName = country.languages.map(language => {
          var langObj = {
            nome: language.name,
            valor: language.iso639_1
          }
          return langObj
        })

        return languageName
      })
      const langs = []

      language.forEach(item => {
        item.forEach(item => {
          if (langs.indexOf(item) === -1) {
            langs.push(item)
          }else {
            console.log('error');
          }
        });
      })
      console.log(langs);
      return langs

    },
    countryName(){
      var countryName = this.countries.map(country => {

        return {
          nome: country.name,
          valor: country.name
        }
      })

      return countryName

    },
    callingCode(){
      var callingCode = this.countries.map(country => {
        var code = country.callingCodes.toString()
        return {
          nome: code,
          valor: code
        }
      })

      return callingCode

    },
    secondFilter(){

      if (this.firstFilter === 'capital') {
        return this.capitals
      }
      else if(this.firstFilter === 'lang'){
        return this.language
      }
      else if(this.firstFilter === 'name'){
        return this.countryName
      }
      else if(this.firstFilter === 'region'){
        return this.continents
      }
      else if(this.firstFilter === 'callingcode'){
        return this.callingCode
      }
    }
  },
  mounted(){
    this.getCountries()
  }
}

</script>

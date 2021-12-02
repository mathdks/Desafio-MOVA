<template>
    <div>
        <v-container>
            <v-row v-for="country in countries" :key="country.name">
                <v-img 
                    :src="country.flag"
                    class="col-4"
                />
                <div>
                    <p>Nome: {{country.name}}</p>
                    <p>Capital: {{country.capital}}</p>
                    <p>Região: {{country.region}}</p>
                    <p>Sub-Região: {{country.subregion}}</p>
                    <p>População: {{country.population}}</p>
                    <p>Línguas: {{langs}}</p>
                </div>
            </v-row>
        </v-container>
        <v-container>
            <v-row>
                <v-container v-for="country in countries" :key="country.name">
                    <p v-if="borders.length > 0">
                        Países Vizinhos
                    </p>
                    <p v-else>
                        {{country.name}} é uma ilha e, devido a isso, não faz fronteira com nenhum outro país.
                    </p>
                </v-container>
                <v-container>
                    <v-row>
                        <v-col v-for="border in bordersList" :key="border.flag">
                            <nuxt-link :to="`/${border.name}`">
                                <v-img
                                :lazy-src="border.flag"
                                :src="border.flag"
                                class="col-6"
                                max-height="150"
                                max-width="250"
                                />
                            </nuxt-link>
                        </v-col>
                    </v-row>
                </v-container>
            </v-row>
        </v-container>
        <v-container>
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
    data(){
        return {
            page: 1,
            totalPages: null,
            pageSize: 12,
            countries: [],
            borders: [],
            bordersList: []
        }
    },
    methods: {
        async getCountries(){
            var slug = this.$route.params.slug
            const countries = await this.$axios.$get(`/name/${slug}`)
            this.countries = countries

            
            var countryBorders = this.countries[0].borders
            countryBorders.forEach(async (country) => {
                const countries = await this.$axios.$get(`/alpha/${country}`)
                
                this.borders.push(countries)

                this.totalPages = Math.ceil(this.borders.length / this.pageSize)

                if (this.bordersList.length < 12) {
                    
                    this.bordersList.push(countries)
                }
                
            })
        },
        next(){
            if (this.page === 1) {
                this.bordersList = this.borders.slice(0, this.page*12)
            }else{
                this.bordersList = this.borders.slice(this.page*12, this.page*12+12)
            }
        }
    },
    computed: {
        langs(){
            var language = this.countries.map(country => {
                var languageName = country.languages.map(language => {
                    
                    return language.name
                })

                return languageName.toString()
            })

            return language.toString()
        }
    },
    created(){
        this.getCountries()
    }
}
</script>

<style>

</style>
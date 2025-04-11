<template>
  <v-app>
    <v-container>
      <v-container>
        <v-row dense>
          <!-- Filtros -->
          <v-col cols="12" sm="4">
            <v-text-field
              label="Filtrar por ID ou nome (ex: 4, Pikachu)"
              v-model="search"
              @keyup="searchPokemon"
              solo
              dense
            />
          </v-col>

          <v-col cols="12" sm="4">
            <v-text-field
              label="Filtrar por tipo (ex: fire, grass, water...)"
              v-model="searchType"
              @keyup="searchPokemon"
              solo
              dense
            />
          </v-col>

          <v-col cols="12" sm="4">
            <v-text-field
              label="Filtrar por espécie (ex: pikachu, bulbasaur...)"
              v-model="searchSpecies"
              @keyup="searchPokemon"
              solo
              dense
            />
          </v-col>
        </v-row>

        <!-- Lista de pokémons -->
        <v-row>
          <template v-if="Object.entries(pokemonData).length > 0">
            <v-col cols="12" sm="3">
              <v-card @click="show_pokemon(pokemonData.id)">
                <v-container>
                  <v-row class="mx-0 d-flex justify-center">
                    <img
                      :src="`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${pokemonData.id}.png`"
                      :alt="pokemonData.name"
                      width="50%"
                    />
                  </v-row>
                  <h2 class="text-center">{{ get_name(pokemonData) }}</h2>
                </v-container>
              </v-card>
            </v-col>
          </template>

          <template v-else>
            <v-col
              cols="12"
              sm="3"
              v-for="pokemon in filtered_pokemons"
              :key="pokemon.name"
            >
              <v-card @click="show_pokemon(get_id(pokemon))">
                <v-container>
                  <v-row class="mx-0 d-flex justify-center">
                    <img
                      :src="`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${get_id(pokemon)}.png`"
                      :alt="pokemon.name"
                      width="50%"
                    />
                  </v-row>
                  <h2 class="text-center">{{ get_name(pokemon) }}</h2>
                </v-container>
              </v-card>
            </v-col>
          </template>
        </v-row>
      </v-container>
    </v-container>

    <!-- Modal Detalhes do Pokémon -->
    <v-dialog v-model="show_dialog" width="600">
      <v-card v-if="selected_pokemon" class="px-6 py-6">
        <v-container>
          <v-row class="d-flex align-center">
            <v-col cols="4">
              <img
                :src="`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${selected_pokemon.id}.png`"
                :alt="selected_pokemon.name"
                width="100%"
              />
            </v-col>

            <v-col cols="8">
              <h1>{{ get_name(selected_pokemon) }}</h1>

              <v-divider class="my-4"></v-divider>

              <v-chip
                class="mr-2"
                v-for="type in selected_pokemon.types"
                :key="type.slot"
                :class="`type-${type.type.name}`"
              >
                {{ type.type.name }}
              </v-chip>

              <v-divider class="my-4"></v-divider>

              <v-row dense class="d-flex flex-wrap" style="gap: 12px 8px">
                <v-chip>ID: {{ selected_pokemon.id }}</v-chip>
                <v-chip>Altura: {{ selected_pokemon.height * 2.54 }} cm</v-chip>
                <v-chip>Peso: {{ (selected_pokemon.weight * 0.45359).toFixed(0) }} kg</v-chip>
              </v-row>
            </v-col>
          </v-row>

          <!-- Tabela de movimentos -->
          <h2 class="my-6">Ataques (Moves)</h2>
          <v-simple-table>
            <thead>
              <tr>
                <th class="text-left">Nivel (Level)</th>
                <th class="text-left">Nome (Name)</th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="item in filter_moves(selected_pokemon)"
                :key="item.move.name"
              >
                <td>{{ get_level(item) }}</td>
                <td>{{ item.move.name }}</td>
              </tr>
            </tbody>
          </v-simple-table>

          <!-- Jogos -->
          <h2 class="my-6 ml-2">Presente nos Jogos</h2>
          <v-row dense class="d-flex flex-wrap ml-2" style="gap: 12px 8px">
            <v-chip
              v-for="entry in selected_pokemon.game_indices"
              :key="entry.version.name"
            >
              {{ entry.version.name }}
            </v-chip>
          </v-row>
        </v-container>
      </v-card>
    </v-dialog>
  </v-app>
</template>

<script>
import axios from 'axios';

export default {
  name: 'App',
  data() {
    return {
      pokemons: [],
      pokemonData: {},
      search: "",
      searchType: "",
      searchSpecies: "",
      show_dialog: false,
      selected_pokemon: null,
      pokeapi: 'https://pokeapi.co/api/v2/pokemon',
    };
  },
  mounted() {
    this.loadPokemons();
  },
  methods: {
    async loadPokemons() {
      try {
        const response = await axios.get(`${this.pokeapi}?limit=1000`);
        this.pokemons = response.data.results;
      } catch (error) {
        console.error("Erro ao carregar pokémons:", error);
      }
    },
    get_id(pokemon) {
      return Number(pokemon.url?.split("/")[6] || pokemon.id);
    },
    get_name(pokemon) {
      return pokemon.name.charAt(0).toUpperCase() + pokemon.name.slice(1);
    },
    get_level(move) {
      for (let versiongroup of move.version_group_details) {
        if (
          versiongroup.version_group.name === "sword-shield" &&
          versiongroup.move_learn_method.name === "level-up"
        ) {
          return versiongroup.level_learned_at;
        }
      }
      return 0;
    },
    show_pokemon(id) {
      axios.get(`${this.pokeapi}/${id}`).then((response) => {
        this.selected_pokemon = response.data;
        this.show_dialog = true;
      });
    },
    async searchPokemon() {
      this.pokemonData = {};
      const searchValue = this.search.trim().toLowerCase();
      const typeValue = this.searchType.trim().toLowerCase();
      const speciesValue = this.searchSpecies.trim().toLowerCase();

      try {
        if (searchValue) {
          const response = await axios.get(`${this.pokeapi}/${searchValue}`);
          this.pokemonData = response.data;
          return;
        }

        if (typeValue) {
          const response = await axios.get(`https://pokeapi.co/api/v2/type/${typeValue}`);
          const pokemons = response.data.pokemon.map(p => p.pokemon);
          this.pokemons = pokemons;
          return;
        }

        if (speciesValue) {
          const speciesResponse = await axios.get(`https://pokeapi.co/api/v2/pokemon-species/${speciesValue}`);
          const pokemonUrl = speciesResponse.data.varieties[0].pokemon.url;
          const pokemonResponse = await axios.get(pokemonUrl);
          this.pokemonData = pokemonResponse.data;
          return;
        }

        await this.loadPokemons();

      } catch (error) {
        console.error("Erro na busca:", error);
        this.pokemonData = {};
      }
    },
    filter_moves(pokemon) {
      return pokemon.moves.filter((item) => {
        return item.version_group_details.some(
          (versiongroup) =>
            versiongroup.version_group.name === "sword-shield" &&
            versiongroup.move_learn_method.name === "level-up"
        );
      });
    },
  },
  computed: {
    filtered_pokemons() {
      return this.pokemons.filter((item) =>
        item.name.includes(this.search.toLowerCase())
      );
    },
  },
};
</script>

<style lang="scss">
@import './pokemon_types.scss';

#app {
  background: rgb(0, 0, 0);
  background-size: cover !important;
  background-position: center;
  min-height: 100vh;
}
</style>

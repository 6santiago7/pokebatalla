<template>
  <div class="app">
    <div v-if="rounds === 0" class="round-selection">
      <img src="https://i.pinimg.com/originals/8d/66/d9/8d66d96a9893ee18763d913767db869f.png" alt="Seleccionar Rondas" class="header-image" />
      <h2>Selecciona el número de rondas:</h2>
      <button @click="selectRounds(3)">3 Rondas</button>
      <button @click="selectRounds(5)">5 Rondas</button>
      <button @click="selectRounds(7)">7 Rondas</button>

      <h2>Selecciona el modo de combate:</h2>
      <select v-model="selectedMode">
        <option value="attack">Ataque</option>
        <option value="defense">Defensa</option>
        <option value="speed">Velocidad</option>
      </select>
    </div>

    <div v-if="rounds > 0 && !gameOver" class="battle">
      <h2>Ronda {{ currentRound }}</h2>

      <div class="scoreboard">
        <p>Equipo 1 - Victorias: {{ team1Wins }}</p>
        <p>Equipo 2 - Victorias: {{ team2Wins }}</p>
      </div>

      <div class="team">
        <h3>Equipo 1</h3>
        <ul class="pokemon-cards">
          <li
            v-for="(pokemon, index) in team1"
            :key="pokemon.name"
            :class="{'selected-pokemon': selectedPokemon1 === pokemon}" 
            class="pokemon-card"
          >
            <img :src="pokemon.image" :alt="pokemon.name" class="pokemon-image" />
            <button class="select-btn" @click="selectPokemon(1, index)">
              {{ pokemon.name }} (Stats: {{ pokemonStats(pokemon) }})
            </button>
          </li>
        </ul>
      </div>

      <div class="team">
        <h3>Equipo 2</h3>
        <ul class="pokemon-cards">
          <li
            v-for="(pokemon, index) in team2"
            :key="pokemon.name"
            :class="{'selected-pokemon': selectedPokemon2 === pokemon}" 
            class="pokemon-card"
          >
            <img :src="pokemon.image" :alt="pokemon.name" class="pokemon-image" />
            <button class="select-btn" @click="selectPokemon(2, index)">
              {{ pokemon.name }} (Stats: {{ pokemonStats(pokemon) }})
            </button>
          </li>
        </ul>
      </div>

      <div v-if="selectedPokemon1 && selectedPokemon2" class="battle-result">
        <h2>Batalla: {{ selectedPokemon1.name }} vs {{ selectedPokemon2.name }}</h2>
        <p>{{ selectedPokemon1.name }} Stats: {{ pokemonStats(selectedPokemon1) }}</p>
        <p>{{ selectedPokemon2.name }} Stats: {{ pokemonStats(selectedPokemon2) }}</p>
        <button class="battle-btn" @click="battle">Comenzar batalla</button>
      </div>
    </div>

    <div v-if="winner !== null && !gameOver" class="results">
      <h2>Ganador de la ronda: Equipo {{ winner }}</h2>
      <button @click="nextRound">Siguiente Ronda</button>
    </div>

    <div v-if="gameOver" class="game-over">
      <h2>¡Fin de la batalla! El ganador es el Equipo {{ finalWinner }}.</h2>
      <p>Puntuación Final:</p>
      <p>Equipo 1: {{ team1Wins }} Victorias</p>
      <p>Equipo 2: {{ team2Wins }} Victorias</p>
      <button @click="startNewGame" class="new-game-btn">Nueva Batalla</button>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";

export default {
  setup() {
    const allPokemon = ref([]);
    const team1 = ref([]);
    const team2 = ref([]);
    const rounds = ref(0);
    const currentRound = ref(1);
    const team1Wins = ref(0);
    const team2Wins = ref(0);
    const selectedPokemon1 = ref(null);
    const selectedPokemon2 = ref(null);
    const winner = ref(null);
    const gameOver = ref(false);
    const finalWinner = ref(null);
    const selectedMode = ref("attack");

    const fetchPokemon = async () => {
      const response = await fetch("https://pokeapi.co/api/v2/pokemon?limit=150");
      const data = await response.json();
      const pokemonData = await Promise.all(
        data.results.map(async (pokemon) => {
          const details = await fetch(pokemon.url);
          const detailsData = await details.json();
          return {
            name: pokemon.name,
            image: detailsData.sprites.front_default,
            stats: detailsData.stats,
          };
        })
      );
      allPokemon.value = pokemonData;
    };

    const selectRounds = (numRounds) => {
      rounds.value = numRounds;
      createTeams();
    };

    const createTeams = () => {
      team1.value = [];
      team2.value = [];
      selectedPokemon1.value = null;
      selectedPokemon2.value = null;
      for (let i = 0; i < 4; i++) {
        const randomPokemon1 = allPokemon.value[Math.floor(Math.random() * allPokemon.value.length)];
        const randomPokemon2 = allPokemon.value[Math.floor(Math.random() * allPokemon.value.length)];
        team1.value.push(randomPokemon1);
        team2.value.push(randomPokemon2);
      }
    };

    const pokemonStats = (pokemon) => {
      switch (selectedMode.value) {
        case "attack":
          return pokemon.stats[1].base_stat; 
        case "defense":
          return pokemon.stats[2].base_stat; 
        case "speed":
          return pokemon.stats[5].base_stat; 
        default:
          return 0;
      }
    };

    const selectPokemon = (team, index) => {
      if (team === 1) {
        selectedPokemon1.value = team1.value[index];
      } else {
        selectedPokemon2.value = team2.value[index];
      }
    };

    const battle = () => {
      const team1Stats = pokemonStats(selectedPokemon1.value);
      const team2Stats = pokemonStats(selectedPokemon2.value);

      if (team1Stats > team2Stats) {
        team1Wins.value++;
        winner.value = 1;
      } else {
        team2Wins.value++;
        winner.value = 2;
      }

      if (team1Wins.value >= Math.ceil(rounds.value / 2)) {
        gameOver.value = true;
        finalWinner.value = 1;
      } else if (team2Wins.value >= Math.ceil(rounds.value / 2)) {
        gameOver.value = true;
        finalWinner.value = 2;
      }
    };

    const nextRound = () => {
      winner.value = null;
      selectedPokemon1.value = null;
      selectedPokemon2.value = null;
      currentRound.value++;

      if (!gameOver.value) {
        createTeams();
      }
    };

    const startNewGame = () => {
      rounds.value = 0;
      currentRound.value = 1;
      team1Wins.value = 0;
      team2Wins.value = 0;
      winner.value = null;
      gameOver.value = false;
      finalWinner.value = null;
      selectedMode.value = "attack"; 
    };

    onMounted(() => {
      fetchPokemon();
    });

    return {
      team1,
      team2,
      rounds,
      currentRound,
      team1Wins,
      team2Wins,
      selectedPokemon1,
      selectedPokemon2,
      winner,
      gameOver,
      finalWinner,
      selectedMode,
      selectRounds,
      selectPokemon,
      nextRound,
      pokemonStats,
      battle,
      startNewGame,
    };
  },
};
</script>

<style>

.app {
  text-align: center;
  background-size: cover;
  background-position: center;
  color: white;
}

.round-selection {
  margin-top: 80px;
}

.header-image {
  width: 200px;
  height: auto;
  margin-bottom: 20px;
}

.round-selection button {
  margin: 10px;
  padding: 10px 20px;
  font-size: 16px;
  background-color: #48c9b0;
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 8px;
  transition: background-color 0.3s ease;
}

.round-selection button:hover {
  background-color: #36b39e;
}

.pokemon-card {
  display: inline-block;
  margin: 10px;
  border: 2px solid #2980b9;
  border-radius: 8px;
  overflow: hidden;
  transition: transform 0.2s ease;
}

.pokemon-card:hover {
  transform: scale(1.05);
}

.pokemon-image {
  width: 100px;
  height: 100px;
}

.select-btn {
  margin: 5px;
  padding: 5px 10px;
  font-size: 14px;
  background-color: #3498db;
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 5px;
  transition: background-color 0.3s ease;
}

.select-btn:hover {
  background-color: #2980b9;
}

.selected-pokemon {
  border: 2px solid #2ecc71;
}

.battle-result {
  margin-top: 20px;
}

.battle-btn {
  margin-top: 10px;
  padding: 10px 20px;
  font-size: 16px;
  background-color: #e67e22;
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 8px;
  transition: background-color 0.3s ease;
}

.battle-btn:hover {
  background-color: #d35400;
}

.results {
  margin-top: 20px;
}

.game-over {
  margin-top: 20px;
}

.new-game-btn {
  padding: 10px 20px;
  font-size: 16px;
  background-color: #e74c3c;
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 8px;
  transition: background-color 0.3s ease;
}

.new-game-btn:hover {
  background-color: #c0392b;
}
</style>

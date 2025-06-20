<template>
  <div class="search-page-container">
    <div class="search-card">
      <h1 class="mb-4 text-center">Recipe Search</h1>
      <form @submit.prevent="onSearch" class="mb-3">
        <div class="search-vertical">
          <div class="search-box">
            <label class="form-label">Recipe Name</label>
            <input v-model="searchQuery" type="text" class="form-control" placeholder="Search for a recipe..." />
          </div>
          <div class="search-box">
            <label class="form-label">Number of Results</label>
            <select v-model.number="resultsCount" class="form-select">
              <option :value="5">5</option>
              <option :value="10">10</option>
              <option :value="15">15</option>
            </select>
          </div>
          <div class="search-box">
            <label class="form-label">Cuisine Filter</label>
            <select v-model="cuisineInput" class="form-select">
              <option value="">No filter</option>
              <option v-for="c in cuisines" :key="c" :value="c">{{ c }}</option>
            </select>
          </div>
          <div class="search-box">
            <label class="form-label">Diet Filter</label>
            <select v-model="dietInput" class="form-select">
              <option value="">No filter</option>
              <option v-for="d in diets" :key="d" :value="d">{{ d }}</option>
            </select>
          </div>
          <div class="search-box">
            <label class="form-label">Intolerance Filter</label>
            <select v-model="intoleranceInput" class="form-select">
              <option value="">No filter</option>
              <option v-for="i in intolerances" :key="i" :value="i">{{ i }}</option>
            </select>
          </div>
          <div class="search-box">
            <label class="form-label">Sort by:</label>
            <select v-model="sortBy" class="form-select">
              <option value="">No sorting</option>
              <option value="readyInMinutes">Preparation Time</option>
              <option value="popularity">Popularity</option>
            </select>
          </div>
          <div class="search-box align-self-end">
            <button type="submit" class="btn btn-primary w-100">Search</button>
          </div>
        </div>
      </form>
    </div>

    <div v-if="showLastSearch && lastSearchResults.length" class="mb-4">
      <h5 class="text-center">Your Last Search:</h5>
      <div class="row justify-content-center">
        <div class="col-md-4 mb-3" v-for="recipe in lastSearchResults" :key="recipe.id">
          <RecipePreview :recipe="recipe" />
        </div>
      </div>
    </div>

    <div v-if="loading" class="text-center my-5">
      <div class="spinner-border" role="status"><span class="visually-hidden">Loading...</span></div>
    </div>

    <div v-if="!loading && searchResults.length">
      <div class="d-flex justify-content-between align-items-center mb-2">
        <h5>Search Results ({{ searchResults.length }})</h5>
      </div>
      <div class="row justify-content-center">
        <div class="col-md-4 mb-3" v-for="recipe in sortedResults" :key="recipe.id">
          <RecipePreview :recipe="recipe" />
        </div>
      </div>
    </div>
    <div v-if="!loading && searchResults.length === 0 && searchedOnce" class="alert alert-warning mt-4 text-center">
      No matching results found.
    </div>
  </div>
</template>

<script>
import RecipePreview from '../components/RecipePreview.vue';
import store from '@/store';
import axios from 'axios';

const cuisines = [
  "African", "Asian", "American", "British", "Cajun", "Caribbean", "Chinese", "Eastern European", "European", "French", "German", "Greek", "Indian", "Irish", "Italian", "Japanese", "Jewish", "Korean", "Latin American", "Mediterranean", "Mexican", "Middle Eastern", "Nordic", "Southern", "Spanish", "Thai", "Vietnamese"
];
const diets = [
  "Gluten Free", "Ketogenic", "Vegetarian", "Lacto-Vegetarian", "Ovo-Vegetarian", "Vegan", "Pescetarian", "Paleo", "Primal", "Low FODMAP", "Whole30"
];
const intolerances = [
  "Dairy", "Egg", "Gluten", "Grain", "Peanut", "Seafood", "Sesame", "Shellfish", "Soy", "Sulfite", "Tree Nut", "Wheat"
];

export default {
  name: 'SearchPage',
  components: { RecipePreview },
  data() {
    return {
      searchQuery: '',
      resultsCount: 5,
      sortBy: '',
      searchResults: [],
      loading: false,
      searchedOnce: false,
      lastSearchResults: [],
      showLastSearch: false,
      cuisineInput: '',
      dietInput: '',
      intoleranceInput: '',
      cuisines,
      diets,
      intolerances,
    };
  },
  computed: {
    sortedResults() {
      if (!this.sortBy) return this.searchResults;
      return [...this.searchResults].sort((a, b) => {
        if (this.sortBy === 'readyInMinutes') {
          return (a.readyInMinutes || 0) - (b.readyInMinutes || 0);
        }
        if (this.sortBy === 'popularity') {
          return (b.popularity || 0) - (a.popularity || 0);
        }
        return 0;
      });
    }
  },
  created() {
    // Show last search if user is logged in and last search exists
    if (store.username && store.username.value) {
      const last = localStorage.getItem('lastSearchResults');
      if (last) {
        try {
          this.lastSearchResults = JSON.parse(last);
          this.showLastSearch = true;
        } catch {}
      }
    }
  },
  methods: {
    async onSearch() {
      this.loading = true;
      this.searchedOnce = true;
      this.searchResults = [];
      this.showLastSearch = false;
      try {
        const params = {
          query: this.searchQuery,
          number: this.resultsCount,
        };
        if (this.cuisineInput) params.cuisine = this.cuisineInput;
        if (this.dietInput) params.diet = this.dietInput;
        if (this.intoleranceInput) params.intolerance = this.intoleranceInput;
        const response = await axios.get(`${store.server_domain}/recipes/searchrecipe`, { params });
        this.searchResults = response.data.results || [];
        // Save last search for logged-in users
        if (store.username && store.username.value) {
          localStorage.setItem('lastSearchResults', JSON.stringify(this.searchResults));
        }
      } catch (e) {
        this.searchResults = [];
      } finally {
        this.loading = false;
      }
    }
  }
};
</script>

<style scoped>
.title {
  margin-bottom: 2rem;
}

.search-page-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  background: #f8fafc;
  padding-top: 40px;
}

.search-card {
  background: #fff;
  border-radius: 18px;
  box-shadow: 0 4px 24px rgba(0,0,0,0.08);
  padding: 2.5rem 2rem 2rem 2rem;
  max-width: 500px;
  width: 100%;
  margin-bottom: 2.5rem;
}

form .form-label {
  font-weight: 500;
}

.search-vertical {
  display: flex;
  flex-direction: column;
  gap: 18px;
}

.search-box {
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
}

.search-box .form-label,
.search-box .form-check-label {
  margin-bottom: 4px;
}

.search-box .form-select,
.search-box .form-control {
  min-height: 38px;
}

.align-self-end {
  align-self: flex-end;
}

.btn-primary {
  background: #007bff;
  border: none;
  font-weight: 600;
  letter-spacing: 1px;
}

.btn-primary:hover {
  background: #0056b3;
}
</style>
  
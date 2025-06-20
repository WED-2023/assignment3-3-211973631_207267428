<template>
  <div class="container mt-4">
    <h1>My Family Recipes</h1>
    <div v-if="loading" class="text-center my-4">
      <span>Loading family recipes...</span>
    </div>
    <div v-else>
      <div v-if="recipes.length === 0" class="alert alert-info mt-4">
        No family recipes found.
      </div>
      <div v-else>
        <div v-for="(recipe, idx) in recipes" :key="idx" class="card mb-4">
          <div class="card-body">
            <h4 class="card-title">{{ recipe.name }}</h4>
            <h6 class="card-subtitle mb-2 text-muted">By: {{ recipe.recipe_owner }}</h6>
            <h6 class="card-subtitle mb-2 text-muted">When it's made: {{ recipe.when_its_made }}</h6>
            <div v-if="recipe.image" class="mb-3 text-center">
              <img :src="recipe.image" alt="Family Recipe Image" style="max-width: 300px; max-height: 200px; border-radius: 8px;" />
            </div>
            <h5>Ingredients</h5>
            <p style="white-space: pre-line;">{{ recipe.ingredients }}</p>
            <h5>Instructions</h5>
            <p style="white-space: pre-line;">{{ recipe.instructions }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import store from '@/store';
export default {
  name: 'MyFamilyRecipes',
  data() {
    return {
      recipes: [],
      loading: true,
    };
  },
  async mounted() {
    this.loading = true;
    try {
      const res = await axios.get(`${store.server_domain}/users/familyrecipes`, { withCredentials: true });
      this.recipes = Array.isArray(res.data) ? res.data : [];
    } catch (e) {
      this.recipes = [];
    }
    this.loading = false;
  }
};
</script>

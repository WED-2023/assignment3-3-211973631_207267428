<template>
  <div class="card recipe-card" @click="goToRecipePage">
    <div class="image-wrapper">
      <img :src="recipe.image" class="card-img-top recipe-image" :alt="recipe.title" />
      <div class="watched-indicator">
        <span v-if="isSeen" class="badge bg-secondary">👁️ Seen</span>
      </div>
      <div class="hover-overlay">Click to view recipe</div>
    </div>
    <div class="card-body">
      <h5 class="card-title">{{ recipe.title }}</h5>
      <p class="card-text">
        Preparation time: {{ displayMinutes }} minutes<br />
        <span :class="['like-count', likeFlash ? 'like-flash' : '']">
          Popularity: {{ displayLikes }} Likes
        </span>
      </p>
      <div class="badges mb-2">
        <span v-if="recipe.vegan" class="badge bg-success">Vegan</span>
        <span v-if="recipe.vegetarian" class="badge bg-primary">Vegetarian</span>
        <span v-if="recipe.glutenFree" class="badge bg-warning text-dark">Gluten Free</span>
        <span v-else class="badge bg-danger text-light">Contains Gluten</span>
      </div>
      <button
        :class="['btn', isFavorite ? 'btn-danger' : 'btn-outline-danger', 'w-100', 'favorite-btn']"
        @click.stop="addToFavorites"
      >
        <span v-if="isFavorite">❤️ In Favorites</span>
        <span v-else>♡ Add to Favorites</span>
      </button>
      <button
        class="btn btn-outline-success w-100 mt-2 like-btn"
        @click.stop="likeRecipe"
        :disabled="isLiked"
      >
        👍 I liked it
      </button>
    </div>
  </div>
</template>

<script>
import { useRouter } from 'vue-router';
export default {
  name: "RecipePreview",
  props: {
    recipe: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      optimisticFavorite: false,
      optimisticLikes: 0,
      likeFlash: false,
    };
  },
  computed: {
    store() {
      return this.$root.store;
    },
    isFavorite() {
      if (this.optimisticFavorite) return true;
      return this.store.favoriteRecipeIds.includes(this.recipe.id);
    },
    isSeen() {
      return this.store.lastWatchedRecipeIds.includes(this.recipe.id);
    },
    displayMinutes() {
      const min = Number(this.recipe.readyInMinutes);
      return isNaN(min) ? 0 : min;
    },
    displayLikes() {
      return (this.recipe.popularity || 0) + this.optimisticLikes;
    },
    isLiked() {
      return this.store.likedRecipeIds.includes(this.recipe.id);
    },
  },
  methods: {
    goToRecipePage() {
      const router = this.$router || (this.$.appContext && this.$.appContext.config.globalProperties.$router);
      // Optimistically mark as watched if logged in
      if (this.store.username && this.store.username.value) {
        // Add to start of array if not already present
        const idx = this.store.lastWatchedRecipeIds.indexOf(this.recipe.id);
        if (idx !== -1) {
          this.store.lastWatchedRecipeIds.splice(idx, 1);
        }
        this.store.lastWatchedRecipeIds.unshift(this.recipe.id);
        // Keep only last 10 for UI
        if (this.store.lastWatchedRecipeIds.length > 10) {
          this.store.lastWatchedRecipeIds.splice(10);
        }
        // Notify backend (not awaited)
        this.axios.post(
          this.store.server_domain + "/users/lastwatched",
          { amount: -1 },
          { withCredentials: true }
        ).catch(() => {});
      }
      if (router) {
        router.push(`/recipe/${this.recipe.id}`);
      } else {
        console.error('Router instance not found');
      }
    },
    async addToFavorites() {
      if (this.isFavorite) return;
      this.optimisticFavorite = true;
      this.store.favoriteRecipeIds.push(this.recipe.id);
      try {
        await this.axios.post(
          this.$root.store.server_domain + "/users/favorites",
          { recipeId: this.recipe.id },
          { withCredentials: true }
        );
      } catch (error) {
        this.optimisticFavorite = false;
        const idx = this.store.favoriteRecipeIds.indexOf(this.recipe.id);
        if (idx !== -1) this.store.favoriteRecipeIds.splice(idx, 1);
        alert("Failed to add to favorites.");
      }
    },
    async likeRecipe() {
      if (this.isLiked) return;
      this.optimisticLikes += 1;
      this.likeFlash = true;
      setTimeout(() => { this.likeFlash = false; }, 400);
      try {
        await this.axios.post(
          this.store.server_domain + "/users/userlikes",
          { recipeId: this.recipe.id },
          { withCredentials: true }
        );
        this.store.addLiked(this.recipe.id);
      } catch (error) {
        this.optimisticLikes -= 1;
        alert('Error sending like');
      }
    },
  },
};
</script>

<style scoped>
.card-link {
  text-decoration: none;
  color: inherit;
  display: block;
}

.recipe-card {
  cursor: pointer;
  transition: transform 0.2s ease;
}

.recipe-card:hover {
  transform: scale(1.02);
}

.image-wrapper {
  position: relative;
}

.recipe-image {
  width: 100%;
  height: 200px;
  object-fit: cover;
  border-radius: 0.5rem 0.5rem 0 0;
}

.watched-indicator {
  position: absolute;
  top: 8px;
  right: 8px;
  z-index: 3;
}

.hover-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.6);
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
  font-weight: 600;
  opacity: 0;
  transition: opacity 0.3s ease;
  z-index: 5;
  pointer-events: none;
  border-radius: 0.5rem 0.5rem 0 0;
}

.recipe-card:hover .hover-overlay {
  opacity: 1;
}

.favorite-btn {
  font-size: 1.1rem;
  font-weight: bold;
  letter-spacing: 1px;
}

.like-count {
  transition: color 0.3s;
}

.like-flash {
  color: #28a745;
  font-weight: bold;
}
</style>

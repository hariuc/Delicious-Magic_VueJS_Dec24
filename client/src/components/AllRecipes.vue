<script setup>
import { ref, onMounted, computed, watch } from "vue";
import { useCatalogStore } from "../stores/catalogStore";
import { useRoute, useRouter } from "vue-router";
import { getAllRecipes } from "../api/recipeApi";
import { formatDate } from "../utils/formatDates";
import { useLoadingStore } from "../stores/loadingStore";
import Loader from "../components/Loader.vue";

const catalogStore = useCatalogStore();
const router = useRouter();
const route = useRoute();
const loadingStore = useLoadingStore();

const foodPage = ref(parseInt(route.query.foodPage) || 1);
const drinkPage = ref(parseInt(route.query.drinkPage) || 1);
const pageSize = 6;

const snackbar = ref({
  show: false,
  message: "",
  color: "red-darken-4",
});

const fetchRecipes = async (type, page) => {
  try {
    loadingStore.setLoading(true);
    const data = await getAllRecipes(type, page, pageSize);
    if (type === "food") {
      catalogStore.foodRecipes = data.recipes;
      catalogStore.foodTotal = data.total;
    } else {
      catalogStore.drinkRecipes = data.recipes;
      catalogStore.drinkTotal = data.total;
    }
  } catch (error) {
    console.error("An error occurred:", error);
    snackbar.value = {
      show: true,
      message: error.message,
      color: "red-darken-4",
    };
  } finally {
    loadingStore.setLoading(false);
  }
};

onMounted(() => {
  fetchRecipes("food", foodPage.value);
  fetchRecipes("drink", drinkPage.value);
});
watch(foodPage, (newPage) => {
  router.push({
    path: "/catalog",
    query: { ...route.query, foodPage: newPage },
  });
  fetchRecipes("food", newPage);
});
watch(drinkPage, (newPage) => {
  router.push({
    path: "/catalog",
    query: { ...route.query, drinkPage: newPage },
  });
  fetchRecipes("drink", newPage);
});

const foodRecipes = computed(() => catalogStore.foodRecipes);
const drinkRecipes = computed(() => catalogStore.drinkRecipes);
</script>

<template>
  <Loader />
  <div class="recipes-all">
    <v-row>
      <v-col>
        <h3>Served at the table</h3>
        <v-row>
          <template v-if="foodRecipes.length">
            <v-col
              v-for="recipe in foodRecipes"
              :key="recipe._id"
              cols="12"
              sm="6"
              md="4"
            >
              <v-card>
                <v-card-item>
                  <v-img
                    :src="recipe.image ? recipe.image : '/recipe-img.png'"
                    alt="Recipe Image"
                    width="65"
                    height="65"
                  ></v-img>
                </v-card-item>
                <v-card-title>{{ recipe.title }}</v-card-title>
                <v-card-subtitle
                  >Time to make: {{ recipe.prepTime }} mins</v-card-subtitle
                >
                <v-card-subtitle>
                  Added on {{ formatDate(recipe.createdAt) }} by
                  <router-link
                    :to="{
                      name: 'user-id',
                      params: { userId: recipe.owner._id },
                    }"
                  >
                    {{ recipe.owner.username }}
                  </router-link>
                </v-card-subtitle>

                <v-card-actions>
                  <v-btn
                    color="amber-darken-2"
                    variant="tonal"
                    @click="
                      () =>
                        router.push({
                          name: 'view-recipe',
                          params: { id: recipe._id },
                        })
                    "
                  >
                    View
                  </v-btn>
                </v-card-actions>
              </v-card>
            </v-col>
          </template>
          <template v-else>
            <h4>No food scrolls to show</h4>
          </template>
        </v-row>
        <v-pagination
          v-model="foodPage"
          :length="Math.ceil(catalogStore.foodTotal / pageSize)"
          color="teal-darken-3"
          class="pagin"
        />
      </v-col>
      <v-divider
        :thickness="2"
        color="teal-darken-3"
        vertical
        class="div-space"
      ></v-divider>
      <v-col>
        <h3>Served at the bar</h3>
        <v-row>
          <template v-if="drinkRecipes.length">
            <v-col
              v-for="recipe in drinkRecipes"
              :key="recipe._id"
              cols="12"
              sm="6"
              md="4"
            >
              <v-card>
                <v-card-item>
                  <v-img
                    :src="recipe.image ? recipe.image : '/recipe-img.png'"
                    alt="Recipe Image"
                    width="65"
                    height="65"
                  ></v-img>
                </v-card-item>
                <v-card-title>{{ recipe.title }}</v-card-title>
                <v-card-subtitle
                  >Time to make: {{ recipe.prepTime }} mins</v-card-subtitle
                >
                <v-card-subtitle>
                  Added on {{ formatDate(recipe.createdAt) }} by
                  <router-link
                    :to="{
                      name: 'user-id',
                      params: { userId: recipe.owner._id },
                    }"
                  >
                    {{ recipe.owner.username }}
                  </router-link>
                </v-card-subtitle>
                <v-card-actions>
                  <v-btn
                    color="amber-darken-2"
                    variant="tonal"
                    @click="
                      () =>
                        router.push({
                          name: 'view-recipe',
                          params: { id: recipe._id },
                        })
                    "
                  >
                    View
                  </v-btn>
                </v-card-actions>
              </v-card>
            </v-col>
          </template>
          <template v-else>
            <h4>No drink scrolls to show</h4>
          </template>
        </v-row>
        <v-pagination
          v-model="drinkPage"
          :length="Math.ceil(catalogStore.drinkTotal / pageSize)"
          color="teal-darken-3"
          class="pagin"
        />
      </v-col>
    </v-row>

    <v-snackbar v-model="snackbar.show" :color="snackbar.color" top>
      {{ snackbar.message }}
    </v-snackbar>
  </div>
</template>

<style scoped>
h2,
h3,
h4 {
  text-align: center;
  margin-bottom: 1.5rem;
}

.v-card-item {
  display: flex;
  align-items: center;
  justify-content: center;
}

.v-row {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

.div-space {
  margin-top: 1.5rem;
}

.pagin {
  padding-top: 0.3rem;
}
</style>

<!-- views/RestaurantList.vue -->
<template>
  <div class="restaurant-list-page">
    <!-- Hero section with search -->
    <section class="hero">
      <div class="container">
        <h1 class="hero__title">Descubre restaurantes increíbles</h1>
        <p class="hero__subtitle">Encuentra tu comida favorita entregada a tu puerta</p>

        <div class="search-container">
          <div class="search-bar">
            <svg class="search-bar__icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor"
              stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <circle cx="11" cy="11" r="8"></circle>
              <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
            </svg>
            <input type="text" v-model="searchQuery" placeholder="Restaurantes, comida, platos..."
              class="search-bar__input">
            <button v-if="searchQuery" @click="searchQuery = ''" class="search-bar__clear">
              <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"
                stroke-linecap="round" stroke-linejoin="round">
                <line x1="18" y1="6" x2="6" y2="18"></line>
                <line x1="6" y1="6" x2="18" y2="18"></line>
              </svg>
            </button>
          </div>
        </div>

        <div class="popular-searches">
          <span class="popular-searches__label">Popular:</span>
          <div class="popular-searches__tags">
            <button v-for="tag in ['Pizza', 'Hamburguesa', 'Sushi', 'Tacos']" :key="tag" @click="searchQuery = tag"
              class="popular-tag">
              {{ tag }}
            </button>
          </div>
        </div>
      </div>
    </section>

    <!-- Categories section -->
    <section class="categories-section">
      <div class="container">
        <div class="categories-scroll">
          <button v-if="showLeftScroll" @click="scrollCategories('left')"
            class="categories-scroll__arrow categories-scroll__arrow--left" aria-label="Ver categorías anteriores">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"
              stroke-linecap="round" stroke-linejoin="round">
              <polyline points="15 18 9 12 15 6"></polyline>
            </svg>
          </button>

          <div class="categories-scroll__container" ref="categoriesContainer">
            <div v-for="category in categories" :key="category.id" class="category-item"
              :class="{ 'category-item--active': selectedCategory === category.id }"
              @click="selectCategory(category.id)">
              <div class="category-item__icon">
                <span v-if="getCategoryIcon(category.id)">{{ getCategoryIcon(category.id) }}</span>
                <svg v-else width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor"
                  stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <circle cx="12" cy="12" r="10"></circle>
                  <line x1="12" y1="8" x2="12" y2="16"></line>
                  <line x1="8" y1="12" x2="16" y2="12"></line>
                </svg>
              </div>
              <span class="category-item__name">{{ category.name }}</span>
            </div>
          </div>

          <button v-if="showRightScroll" @click="scrollCategories('right')"
            class="categories-scroll__arrow categories-scroll__arrow--right" aria-label="Ver más categorías">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"
              stroke-linecap="round" stroke-linejoin="round">
              <polyline points="9 18 15 12 9 6"></polyline>
            </svg>
          </button>
        </div>
      </div>
    </section>

    <!-- Filters section -->
    <section class="filters-section">
      <div class="container">
        <div class="filters-row">
          <div class="sort-filter">
            <span class="filter-label">Ordenar por:</span>
            <div class="select-wrapper">
              <select v-model="sortBy" class="select-control">
                <option value="popularity">Más Popular</option>
                <option value="rating">Mejor Valorados</option>
                <option value="delivery">Tiempo de Entrega</option>
                <option value="price">Precio: Bajo a Alto</option>
              </select>
              <svg class="select-arrow" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor"
                stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <polyline points="6 9 12 15 18 9"></polyline>
              </svg>
            </div>
          </div>

          <div class="price-filter">
            <span class="filter-label">Precio:</span>
            <div class="price-buttons">
              <button v-for="price in ['$', '$$', '$$$', '$$$$']" :key="price" class="price-button"
                :class="{ 'price-button--active': priceFilter.includes(price) }" @click="togglePriceFilter(price)">
                {{ price }}
              </button>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Restaurant list -->
    <section class="restaurants-section">
      <div class="container">
        <!-- Loader -->
        <div v-if="loading" class="loading-container">
          <div class="loading-spinner"></div>
          <p class="loading-text">Buscando los mejores restaurantes para ti</p>
        </div>

        <!-- Empty state -->
        <div v-else-if="filteredRestaurants.length === 0" class="empty-state">
          <div class="empty-state__icon">
            <svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1"
              stroke-linecap="round" stroke-linejoin="round">
              <path d="M22 12H2M16 6l6 6-6 6M8 18l-6-6 6-6"></path>
            </svg>
          </div>
          <h3 class="empty-state__title">No encontramos restaurantes</h3>
          <p class="empty-state__text">Intenta con otros filtros o una nueva búsqueda</p>
          <button @click="resetFilters" class="empty-state__button">Reiniciar filtros</button>
        </div>

        <!-- Results -->
        <div v-else>
          <div class="results-header">
            <h2 class="results-header__title">{{ getResultsTitle() }}</h2>
            <span class="results-header__count">{{ filteredRestaurants.length }} restaurantes encontrados</span>
          </div>

          <div class="restaurant-grid">
            <div v-for="restaurant in filteredRestaurants" :key="restaurant.id" class="restaurant-column">
              <!-- Restaurant Card Component -->
              <div class="restaurant-card" @click="goToRestaurant(restaurant.id)">
                <div class="restaurant-card__image-wrapper">
                  <img :src="restaurant.coverImageUrl" :alt="restaurant.name" class="restaurant-card__image">

                  <div class="restaurant-card__badges">
                    <span v-if="isNew(restaurant)"
                      class="restaurant-card__badge restaurant-card__badge--new">Nuevo</span>
                    <span v-if="isFastDelivery(restaurant)" class="restaurant-card__badge restaurant-card__badge--fast">
                      <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"
                        stroke-linecap="round" stroke-linejoin="round">
                        <path d="M13 2L3 14h9l-1 8 10-12h-9l1-8z"></path>
                      </svg>
                      Rápido
                    </span>
                  </div>

                  <div class="restaurant-card__time-fee">
                    <div class="restaurant-card__delivery-time">
                      <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"
                        stroke-linecap="round" stroke-linejoin="round">
                        <circle cx="12" cy="12" r="10"></circle>
                        <polyline points="12 6 12 12 16 14"></polyline>
                      </svg>
                      {{ restaurant.estimatedDeliveryTime }} min
                    </div>
                  </div>
                </div>

                <div class="restaurant-card__content">
                  <div class="restaurant-card__header">
                    <h3 class="restaurant-card__name">{{ restaurant.name }}</h3>
                    <div class="restaurant-card__rating">
                      <svg width="16" height="16" viewBox="0 0 24 24" fill="#FFCC00" stroke="#FFCC00" stroke-width="0.5"
                        stroke-linecap="round" stroke-linejoin="round">
                        <polygon
                          points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2">
                        </polygon>
                      </svg>
                      <span>{{ restaurant.averageRating }}</span>
                      <span class="restaurant-card__reviews">({{ restaurant.reviewCount || 0 }})</span>
                    </div>
                  </div>

                  <div class="restaurant-card__info">
                    <span>{{ getTipoName(restaurant.tipo) }}</span>
                    <span class="dot-separator"></span>
                    <span>{{ getPriceRangeFromDeliveryFee(restaurant.deliveryFee) }}</span>
                    <span class="dot-separator"></span>
                    <span>{{ restaurant.distance || '1.0' }} km</span>
                  </div>

                  <div class="restaurant-card__delivery-info">
                    <div v-if="restaurant.deliveryFee === 0" class="restaurant-card__free-delivery">
                      <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"
                        stroke-linecap="round" stroke-linejoin="round">
                        <polyline points="20 6 9 17 4 12"></polyline>
                      </svg>
                      Envío gratis
                    </div>
                    <div v-else class="restaurant-card__fee">
                      Envío: ${{ restaurant.deliveryFee.toFixed(2) }}
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div v-if="hasMoreRestaurants" class="load-more">
            <button class="load-more__button">
              <span>Cargar más restaurantes</span>
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"
                stroke-linecap="round" stroke-linejoin="round">
                <polyline points="6 9 12 15 18 9"></polyline>
              </svg>
            </button>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue';
import { useRouter } from 'vue-router';
import { restaurantService } from '@/services/restaurantService';

// Router
const router = useRouter();

// Estado
const restaurants = ref([]);
const loading = ref(true);
const searchQuery = ref('');
const selectedCategory = ref('all');
const sortBy = ref('popularity');
const priceFilter = ref([]);
const hasMoreRestaurants = ref(false);

// Categorías
const categories = [
  { id: 'all', name: 'Todos' },
  { id: 1, name: 'Americana' },
  { id: 2, name: 'Italiana' },
  { id: 3, name: 'Mexicana' },
  { id: 4, name: 'Asiática' },
  { id: 5, name: 'Fast Food' },
  { id: 6, name: 'Saludable' },
  { id: 7, name: 'Postres' },
  { id: 8, name: 'Vegana' }
];

// Iconos para categorías
const getCategoryIcon = (categoryId) => {
  const icons = {
    1: '🍔',
    2: '🍕',
    3: '🌮',
    4: '🍜',
    5: '🍟',
    6: '🥗',
    7: '🍦',
    8: '🥑'
  };
  return icons[categoryId] || '';
};

// Obtener nombre del tipo de restaurante
const getTipoName = (tipo) => {
  const category = categories.find(c => c.id === tipo);
  return category ? category.name : 'Variado';
};

// Calcular rango de precios basado en costo de envío
const getPriceRangeFromDeliveryFee = (fee) => {
  if (fee === 0) return '$';
  if (fee < 2) return '$$';
  if (fee < 4) return '$$$';
  return '$$$$';
};

// Obtener restaurantes - usando el servicio existente
const fetchRestaurants = async () => {
  loading.value = true;
  
  try {
    let data;
    
    // Si hay una categoría seleccionada
    if (selectedCategory.value !== 'all') {
      data = await restaurantService.getRestaurantsByTipo(selectedCategory.value);
    } else {
      // Si no hay filtros, obtener todos los restaurantes
      data = await restaurantService.getAllRestaurants();
    }
    
    restaurants.value = data;
    console.log('Restaurantes cargados correctamente:', data.length);
  } catch (error) {
    console.error('Error al cargar restaurantes:', error);
  } finally {
    loading.value = false;
  }
};

// Filtrar restaurantes (solo por precio, ya que el filtrado por categoría se hace en el backend)
const filteredRestaurants = computed(() => {
  let result = restaurants.value;
  
  // Filtrar por búsqueda localmente
  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase();
    result = result.filter(restaurant => 
      restaurant.name.toLowerCase().includes(query) || 
      (restaurant.description && restaurant.description.toLowerCase().includes(query))
    );
  }
  
  // Filtrar por precio
  if (priceFilter.value.length > 0) {
    result = result.filter(restaurant => {
      const priceRange = getPriceRangeFromDeliveryFee(restaurant.deliveryFee);
      return priceFilter.value.includes(priceRange);
    });
  }
  
  // Ordenar
  if (sortBy.value === 'rating') {
    result = [...result].sort((a, b) => b.averageRating - a.averageRating);
  } else if (sortBy.value === 'delivery') {
    result = [...result].sort((a, b) => a.estimatedDeliveryTime - b.estimatedDeliveryTime);
  } else if (sortBy.value === 'price') {
    result = [...result].sort((a, b) => a.deliveryFee - b.deliveryFee);
  }
  
  return result;
});

// Métodos
const selectCategory = (categoryId) => {
  selectedCategory.value = categoryId;
  fetchRestaurants();
};

const togglePriceFilter = (price) => {
  if (priceFilter.value.includes(price)) {
    priceFilter.value = priceFilter.value.filter(p => p !== price);
  } else {
    priceFilter.value.push(price);
  }
};

const resetFilters = () => {
  searchQuery.value = '';
  selectedCategory.value = 'all';
  sortBy.value = 'popularity';
  priceFilter.value = [];
  fetchRestaurants();
};

const goToRestaurant = (restaurantId) => {
  router.push(`/restaurant/${restaurantId}`);
};

const getResultsTitle = () => {
  if (selectedCategory.value === 'all' && !searchQuery.value) {
    return 'Restaurantes populares';
  } else if (selectedCategory.value !== 'all') {
    const category = categories.find(c => c.id === selectedCategory.value);
    return `Restaurantes ${category ? category.name : ''}`;
  } else if (searchQuery.value) {
    return `Resultados para "${searchQuery.value}"`;
  }
  return 'Restaurantes';
};

// Helpers para badges
const isNew = (restaurant) => {
  if (!restaurant.createdAt) return false;
  const twoMonthsAgo = new Date();
  twoMonthsAgo.setMonth(twoMonthsAgo.getMonth() - 2);
  return new Date(restaurant.createdAt) > twoMonthsAgo;
};

const isFastDelivery = (restaurant) => {
  return restaurant.estimatedDeliveryTime <= 20;
};

// Navegación en carrusel
const categoriesContainer = ref(null);
const showLeftScroll = ref(false);
const showRightScroll = ref(true);

const checkScrollPosition = () => {
  if (!categoriesContainer.value) return;

  const container = categoriesContainer.value;
  showLeftScroll.value = container.scrollLeft > 10;
  showRightScroll.value = container.scrollLeft < (container.scrollWidth - container.clientWidth - 10);
};

const scrollCategories = (direction) => {
  if (!categoriesContainer.value) return;

  const container = categoriesContainer.value;
  const scrollAmount = container.offsetWidth * 0.75;

  if (direction === 'left') {
    container.scrollBy({ left: -scrollAmount, behavior: 'smooth' });
  } else {
    container.scrollBy({ left: scrollAmount, behavior: 'smooth' });
  }

  // Update scroll indicators after scrolling
  setTimeout(checkScrollPosition, 300);
};

// Inicialización al montar
onMounted(() => {
  fetchRestaurants();

  // Verificar indicadores de scroll
  setTimeout(() => {
    if (categoriesContainer.value) {
      checkScrollPosition();
      categoriesContainer.value.addEventListener('scroll', checkScrollPosition);
    }
  }, 300);
});
</script>

<style lang="scss">
// Variables
$primary-color: #06C167; // Color principal de Uber Eats
$black: #000000;
$white: #FFFFFF;
$light-gray: #F6F6F6;
$medium-gray: #EEEEEE;
$dark-gray: #545454;
$text-primary: #000000;
$text-secondary: #757575;
$success-color: #06C167;
$warning-color: #FF8000;
$border-radius-sm: 8px;
$border-radius: 16px;
$box-shadow: 0 4px 16px rgba(0, 0, 0, 0.08);
$transition: all 0.2s ease;

// Container
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 24px;

  @media (max-width: 768px) {
    padding: 0 16px;
  }
}

.restaurant-list-page {
  background-color: $light-gray;
  min-height: 100vh;
  padding-bottom: 60px;
}

// Hero section
.hero {
  background-color: $white;
  padding: 40px 0;
  text-align: center;
  box-shadow: $box-shadow;

  &__title {
    font-size: 32px;
    font-weight: 700;
    margin-bottom: 8px;
    color: $text-primary;

    @media (max-width: 768px) {
      font-size: 24px;
    }
  }

  &__subtitle {
    font-size: 16px;
    color: $text-secondary;
    margin-bottom: 32px;

    @media (max-width: 768px) {
      font-size: 14px;
      margin-bottom: 24px;
    }
  }
}

// Search container
.search-container {
  max-width: 600px;
  margin: 0 auto;
}

// Search bar
.search-bar {
  position: relative;
  display: flex;
  align-items: center;
  background-color: $light-gray;
  border-radius: 24px;
  padding: 0 16px;
  transition: $transition;
  border: 2px solid transparent;

  &:focus-within {
    border-color: $primary-color;
    background-color: $white;
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  }

  &__icon {
    color: $text-secondary;
    margin-right: 12px;
  }

  &__input {
    flex: 1;
    height: 50px;
    border: none;
    background: transparent;
    font-size: 16px;
    color: $text-primary;
    width: 100%;

    &:focus {
      outline: none;
    }

    &::placeholder {
      color: $text-secondary;
    }
  }

  &__clear {
    background: none;
    border: none;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 24px;
    height: 24px;
    border-radius: 50%;
    color: $text-secondary;
    cursor: pointer;

    &:hover {
      background-color: rgba(0, 0, 0, 0.05);
    }
  }
}

// Popular searches
.popular-searches {
  margin-top: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-wrap: wrap;
  gap: 8px;

  &__label {
    font-size: 14px;
    color: $text-secondary;
  }

  &__tags {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
  }
}

// Popular tag
.popular-tag {
  background-color: $light-gray;
  border: none;
  color: $text-primary;
  padding: 8px 16px;
  border-radius: 100px;
  font-size: 14px;
  cursor: pointer;
  transition: $transition;

  &:hover {
    background-color: $medium-gray;
  }
}

// Categories section
.categories-section {
  background-color: $white;
  padding: 16px 0;
  position: sticky;
  top: 0;
  z-index: 100;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  margin-bottom: 24px;
}

// Categories scroll
.categories-scroll {
  position: relative;
  display: flex;
  align-items: center;

  &__container {
    display: flex;
    gap: 12px;
    overflow-x: auto;
    scrollbar-width: none;
    -ms-overflow-style: none;
    padding: 4px 0;
    scroll-behavior: smooth;

    &::-webkit-scrollbar {
      display: none;
    }
  }

  &__arrow {
    position: absolute;
    z-index: 2;
    background-color: $white;
    border: 1px solid $medium-gray;
    color: $text-primary;
    width: 32px;
    height: 32px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    transition: $transition;

    &:hover {
      background-color: $light-gray;
    }

    &--left {
      left: 0;
    }

    &--right {
      right: 0;
    }
  }
}

// Category item
.category-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  min-width: 80px;
  cursor: pointer;
  transition: $transition;

  &__icon {
    width: 56px;
    height: 56px;
    border-radius: 50%;
    background-color: $light-gray;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 24px;
    transition: $transition;
    position: relative;
    overflow: hidden;

    &::after {
      content: '';
      position: absolute;
      inset: 0;
      border-radius: 50%;
      border: 2px solid transparent;
      transition: $transition;
    }
  }

  &__name {
    font-size: 12px;
    font-weight: 500;
    color: $text-secondary;
    transition: $transition;
  }

  &:hover {
    .category-item__icon {
      background-color: $medium-gray;
    }
  }

  &--active {
    .category-item__icon {
      background-color: #EDFCF2;

      &::after {
        border-color: $primary-color;
      }
    }

    .category-item__name {
      color: $primary-color;
      font-weight: 600;
    }
  }
}

// Filters section
.filters-section {
  background-color: $white;
  padding: 16px 0;
  margin-bottom: 24px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

// Filters row
.filters-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 16px;

  @media (max-width: 768px) {
    flex-direction: column;
    align-items: flex-start;
  }
}

// Filter label
.filter-label {
  font-size: 14px;
  color: $text-secondary;
  margin-right: 8px;
}

// Sort filter
.sort-filter {
  display: flex;
  align-items: center;
}

// Select wrapper
.select-wrapper {
  position: relative;
}

// Select control
.select-control {
  appearance: none;
  background-color: $white;
  border: 1px solid $medium-gray;
  border-radius: 8px;
  padding: 8px 32px 8px 12px;
  font-size: 14px;
  color: $text-primary;
  cursor: pointer;

  &:focus {
    outline: none;
    border-color: $primary-color;
  }
}

// Select arrow
.select-arrow {
  position: absolute;
  right: 12px;
  top: 50%;
  transform: translateY(-50%);
  pointer-events: none;
  color: $text-secondary;
}

// Price filter
.price-filter {
  display: flex;
  align-items: center;
}

// Price buttons
.price-buttons {
  display: flex;
  gap: 8px;
}

// Price button
.price-button {
  background-color: $white;
  border: 1px solid $medium-gray;
  color: $text-primary;
  font-size: 14px;
  min-width: 40px;
  height: 40px;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: $transition;

  &:hover {
    background-color: $light-gray;
  }

  &--active {
    background-color: $primary-color;
    border-color: $primary-color;
    color: $white;

    &:hover {
      background-color: $primary-color;
    }
  }
}

// Restaurants section
.restaurants-section {
  padding: 16px 0;
}

// Results header
.results-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;

  @media (max-width: 768px) {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }

  &__title {
    font-size: 24px;
    font-weight: 700;
    color: $text-primary;
    margin: 0;

    @media (max-width: 768px) {
      font-size: 20px;
    }
  }

  &__count {
    color: $text-secondary;
    font-size: 14px;
  }
}

// Restaurant grid
.restaurant-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 24px;

  @media (max-width: 480px) {
    grid-template-columns: 1fr;
  }
}

// Restaurant card
.restaurant-card {
  background-color: $white;
  border-radius: $border-radius;
  overflow: hidden;
  cursor: pointer;
  transition: $transition;
  box-shadow: $box-shadow;
  height: 100%;
  display: flex;
  flex-direction: column;

  &:hover {
    transform: translateY(-4px);
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12);

    .restaurant-card__image {
      transform: scale(1.05);
    }
  }

  &__image-wrapper {
    position: relative;
    height: 176px;
    overflow: hidden;
  }

  &__image {
    width: 100%;
    height: 100%;
    object-fit: cover;
    transition: transform 0.3s ease;
  }

  &__badges {
    position: absolute;
    top: 12px;
    left: 12px;
    display: flex;
    gap: 8px;
  }

  &__badge {
    display: flex;
    align-items: center;
    gap: 4px;
    padding: 4px 8px;
    border-radius: 4px;
    font-size: 12px;
    font-weight: 600;

    &--new {
      background-color: $success-color;
      color: $white;
    }

    &--fast {
      background-color: $warning-color;
      color: $white;
    }
  }

  &__time-fee {
    position: absolute;
    bottom: 12px;
    right: 12px;
  }

  &__delivery-time {
    background-color: rgba(0, 0, 0, 0.7);
    color: $white;
    border-radius: 4px;
    padding: 4px 8px;
    font-size: 12px;
    font-weight: 600;
    display: flex;
    align-items: center;
    gap: 4px;
  }

  &__content {
    padding: 16px;
    flex-grow: 1;
    display: flex;
    flex-direction: column;
  }

  &__header {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    margin-bottom: 8px;
  }

  &__name {
    font-size: 18px;
    font-weight: 700;
    color: $text-primary;
    margin: 0;
  }

  &__rating {
    display: flex;
    align-items: center;
    gap: 4px;
    font-weight: 600;
    font-size: 14px;
    color: $text-primary;
  }

  &__reviews {
    color: $text-secondary;
    font-weight: 400;
    font-size: 13px;
  }

  &__info {
    margin-bottom: 12px;
    color: $text-secondary;
    font-size: 14px;
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    gap: 4px;
  }

  &__delivery-info {
    margin-top: auto;
    font-size: 14px;
  }

  &__free-delivery {
    color: $success-color;
    font-weight: 600;
    display: flex;
    align-items: center;
    gap: 4px;
  }

  &__fee {
    color: $text-secondary;
  }
}

// Dot separator
.dot-separator {
  width: 4px;
  height: 4px;
  background-color: $medium-gray;
  border-radius: 50%;
  display: inline-block;
}

// Loading container
.loading-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 60px 0;

  .loading-spinner {
    width: 48px;
    height: 48px;
    border: 3px solid $light-gray;
    border-radius: 50%;
    border-top-color: $primary-color;
    animation: spinner 1s linear infinite;
    margin-bottom: 16px;
  }

  .loading-text {
    color: $text-secondary;
    font-size: 16px;
  }

  @keyframes spinner {
    to {
      transform: rotate(360deg);
    }
  }
}

// Empty state
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 60px 0;
  text-align: center;

  &__icon {
    width: 80px;
    height: 80px;
    background-color: $light-gray;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    color: $text-secondary;
    margin-bottom: 24px;
  }

  &__title {
    font-size: 20px;
    font-weight: 700;
    color: $text-primary;
    margin: 0 0 8px;
  }

  &__text {
    color: $text-secondary;
    margin: 0 0 24px;
    max-width: 400px;
  }

  &__button {
    background-color: $primary-color;
    color: $white;
    border: none;
    padding: 12px 24px;
    border-radius: 8px;
    font-weight: 600;
    cursor: pointer;
    transition: $transition;

    &:hover {
      background-color: $primary-color;
    }
  }
}

// Load more
.load-more {
  text-align: center;
  margin-top: 40px;

  &__button {
    background-color: transparent;
    border: 1px solid $medium-gray;
    color: $text-primary;
    padding: 12px 24px;
    border-radius: 8px;
    font-weight: 600;
    cursor: pointer;
    display: inline-flex;
    align-items: center;
    gap: 8px;
    transition: $transition;

    &:hover {
      background-color: $light-gray;
    }

    svg {
      transition: transform 0.3s ease;
    }

    &:hover svg {
      transform: translateY(3px);
    }
  }
}
</style>
<template>
  <v-dialog
    fullscreen
    height="95.7vh"
    v-model="dialog"
    activator="parent"
    scroll-strategy="block"
    transition="slide-x-transition"
  >
    <v-card>
      <v-toolbar id="tool-bar" >
        <v-btn icon dark @click="dialog = false">
          <svg-icon class="close-btn" type="mdi" :path="close"></svg-icon>
        </v-btn>
        <v-toolbar-title>
          <div>
            <h3>Cart</h3>
          </div>
        </v-toolbar-title>
        <v-spacer></v-spacer>
        <v-btn id="checkout-btn" @click="checkout" >Checkout</v-btn>
      </v-toolbar>
      <div class="total-wrapper">
        <p>
          You have <span class="count">{{ totalItemsCount }}</span> {{ singularOrPlural }} in your
          cart
        </p>
        <hr />
        <p>
          Total:
          <span>
            <span class="count">{{ totalPriceSum }}</span>
            €
          </span>
        </p>
      </div>
      <div class="cart-item-list d-flex align-content-start justify-center flex-wrap align-end">
        <cart-item
          class="ma-4"
          v-for="cocktail in cartStore.cartItem"
          :key="cocktail.cocktail_name"
          :id="cocktail.cocktail_id"
          :cocktail-name="cocktail.cocktail_name"
          :cocktail-price="cocktail.cocktail_price"
          :amount="cocktail.amount"
          :image-url="cocktail.image_url"
          @increment-item="incrementAmount"
          @decrement-item="decrementAmount"
          @delete-item="deleteItem"
        ></cart-item>
      </div>
    </v-card>
  </v-dialog>
</template>
<script setup>
import { onMounted, computed, ref } from 'vue';
import { useCartStore } from '../../stores/CartStore';
import { useAuthStore } from '../../stores/AuthStore';
import SvgIcon from '@jamescoyle/vue-icon';
import { mdiClose } from '@mdi/js';
import router from '/src/router/routes.js';
let dialog = ref(false);
let close = mdiClose;
const counter = ref(0);
const authStore = useAuthStore();
const cartStore = useCartStore();
const emit = defineEmits(['update:total-items-count']);

onMounted(() => {
  emit('update:total-items-count', totalItemsCount);
});

const checkout = () =>{
  if (cartStore.cartItem.length > 0) {
    cartStore.userWasAboutToCheckout = true;
  }
  if (authStore.isLoggedIn && cartStore.userWasAboutToCheckout) {
    router.push('/checkout');
    closeModal();
  }
  else if (authStore.isLoggedIn && cartStore.cartItem.length === 0) {
    router.push('/cocktails');
    closeModal();
  }
  else {
    router.push('/login');
    closeModal();
  }
}

 const totalPriceSum = computed(() => {
  let totalPrice = 0;
  for (const cartItem of cartStore.cartItem) {
    totalPrice += cartItem.amount * cartItem.cocktail_price;
  }
  return totalPrice;
});
const singularOrPlural = computed(() => {
  if (counter.value === 0 || counter.value > 1) return 'items';
  if (counter.value === 1) return 'item';
});
const totalItemsCount = computed(() => {
  let count = 0;
  for (const cartItem of cartStore.cartItem) {
    count += cartItem.amount;
    counter.value = count;
  }

  emit('update:total-items-count', count);
  return count;
});
function closeModal() {
  dialog.value = false;
  
}
function incrementAmount(cocktailId) {
  const identifiedCocktail = cartStore.cartItem.find(
    cocktail => cocktail.cocktail_id === cocktailId
  );
  identifiedCocktail.amount += 1;
}

function decrementAmount(cocktailId) {
  const identifiedCocktail = cartStore.cartItem.find(
    cocktail => cocktail.cocktail_id === cocktailId
  );
  const identifiedCocktailIndex = cartStore.cartItem.findIndex(
    cocktail => cocktail.cocktail_id === cocktailId
  );
  identifiedCocktail.amount -= 1;
  if (identifiedCocktail.amount === 0) {
    cartStore.cartItem.splice(identifiedCocktailIndex, 1);
  }
}

function deleteItem(cocktailId) {
  const identifiedCocktail = cartStore.cartItem.find(
    cocktail => cocktail.cocktail_id === cocktailId
  );
  cartStore.cartItem.splice(identifiedCocktail, 1);
}
</script>

<style scoped></style>

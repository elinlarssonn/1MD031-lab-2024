<template>

  <div class="burger-item">
    <h3>{{ burger.name }}</h3>
    <ul>
      <li>{{ burger.kCal }} kCal</li>
      <li v-if="burger.gluten">Innehåller <span class="allergen">gluten</span></li>
      <li v-if="burger.lactose">Innehåller <span class="allergen">laktos</span></li>
    </ul>
    <img v-bind:src="burger.image">
    <ul>
      <li v-for="ingredient in burger.ingredients" :key="ingredient">
        {{ ingredient }}
      </li>
    </ul>
    <p>Antal:
      <button @click="decreaseAmount">-</button>
      <span>{{ amountOrdered }}</span>
      <button @click="increaseAmount">+</button>
    </p>
  </div>


</template>

<script>
export default {
  name: 'OneBurger',
  props: {
    burger: Object
  },

  data: function () {
    return {
      amountOrdered: 0,
    }
  },

  methods: {
    increaseAmount() {
      this.amountOrdered++; // Öka antalet
      this.$emit('orderedBurger', {
        name: this.burger.name,
        amount: this.amountOrdered
      }
      );
    },
    decreaseAmount() {
      if (this.amountOrdered > 0) {
        this.amountOrdered--; // Minska antalet, men inte under 0
        this.$emit('orderedBurger', {
        name: this.burger.name,
        amount: this.amountOrdered
        }
       );
      }
    }
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.burger-item {
  display: inline-block;
  padding: 10px;
  /* Skapar lite inre utrymme inom varje hamburgardiv */
  margin: 20px;
  /* Skapar utrymme mellan varje hamburgardiv */
  border: 1px solid #ddd;
  /* Lägg till en ram för att separera varje hamburgare visuellt */
  border-radius: 5px;
  /* Gör hörnen lite rundade (valfritt) */
}

.burger-container {
  display: grid;
  grid-gap: 10px;
  grid-template-columns: repeat(auto-fit, 30%);
}

.burger-container.burger-item {
  border-radius: 5px;
  padding: 20px;
  font-size: 100%;
  width: 100%;
}

.burger-item img {
  width: 100%;
}


</style>
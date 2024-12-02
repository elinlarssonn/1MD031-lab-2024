<template>

  <header class="header-banner">
    <h1>Välkommen till BurgerOnline</h1>
    <img
      src=https://res.cloudinary.com/simpleview/image/upload/v1485793494/clients/chattanooga/Restaurants_cb227c1b-1c14-4eb5-b22f-c67b8dbf4d45.jpg
      alt="Bakgrundsbild" class="header-image">
  </header>

  <section id="burger-selection">
    <h2>Välj din hamburgare</h2>
    <p>Här kan du välja vilken hamburgare du vill beställa.</p>

    <div class="burger-container">
      <Burger v-for="burger in burgers" v-bind:burger="burger" v-bind:key="burger.name"
        v-on:orderedBurger="addToOrder($event)" />
    </div>

  </section>

  <section id="order-info">
    <h2>Leveransinformation</h2>
    <p>Fyll i din leveransadress och kontaktuppgifter för att få din beställning levererad.</p>
    <form>
      <p>
        <label for="fullname">För- och efternamn:</label><br>
        <input type="text" id="fullname" v-model="fn" required="required" placeholder="För- och efternamn">
      </p>
      <p>
        <label for="email">E-mail:</label><br>
        <input type="email" id="email" v-model="em" required="required" placeholder="E-mail adress">
      </p>
    </form>



    <h3>Betalningsalternativ</h3>
    <p>
      <label for="payment-method">Välj betalningsmetod:</label>
      <select id="payment-method" v-model="paymentMethod">
        <option>Kredit kort</option>
        <option>Swish</option>
        <option>Apple pay</option>
      </select>
    </p>


    <h3>Kön</h3>

    <p>Välj ditt kön:
    <div>
      <input type="radio" name="gender" id="man" v-model="gender" value="male">
      <label for="man">Man</label>
    </div>


    <div>
      <input type="radio" name="gender" id="female" v-model="gender" value="female">
      <label for="female">Kvinna</label>
    </div>


    <div>
      <input type="radio" name="gender" id="notprovided" v-model="gender" value="notprovided">
      <label for="notprovided">Vill ej uppge</label>
    </div>
    </p>

    <h3>Leveransadress</h3>
    <p>Vänligen ange din leverensadress:

    <div class="mapSection">
      <div id="dots" v-on:click="setLocation">
        <div v-bind:style="{ left: location.x + 'px', top: location.y + 'px' }">T
        </div>
      </div>
    </div>
    </p>

  </section>


  <button type="button" v-on:click="printValues">
    <img src="https://cdn.pixabay.com/photo/2016/03/31/14/37/check-mark-1292787_1280.png" alt="Beställningsikon"
      style="width: 20px; height: 20px; vertical-align: middle;">
    Skicka Beställning
  </button>

  <hr>

  <footer>
    <p>&copy; 2024 BurgerOnline. Alla rättigheter förbehållna.</p>
  </footer>

</template>

<script>
import Burger from '../components/OneBurger.vue'
import io from 'socket.io-client'
import menu from '../assets/menu.json'

const socket = io("localhost:3000");

function MenuItem(name, img, calories, glu, lac, ingr) {
  this.name = name;
  this.image = img;
  this.kCal = calories;
  this.gluten = glu;
  this.lactose = lac;
  this.ingredients = ingr
}

const b1 = new MenuItem("Cheese Burger", 'https://www.sargento.com/assets/Uploads/Recipe/Image/burger_0.jpg', 500, true, true, ['Nötkött', 'Bacon', 'Gul lök', 'Ost', 'Bröd']);
const b2 = new MenuItem('Chicken Burger', 'https://cdn.kronfagel.se/app/uploads/2023/08/17140546/FS_3179_crispy_burger_HERO-1920x860.jpg', 400, true, true, ['Kyckling', 'Tomat', 'Gurka', 'Dressing', 'Bröd']);
const b3 = new MenuItem('Halloumi Burger', 'https://food-images.files.bbci.co.uk/food/recipes/chargrilled_halloumi_09010_16x9.jpg', 300, true, true, ['Halloumi', 'Tomatsalsa', 'Avocado', 'Sriracha mayo', 'Bröd']);

const items = [b1, b2, b3];

console.log(items);

export default {
  name: 'HomeView',
  components: {
    Burger
  },
  data: function () {
    return {
      burgers: menu,
      orderedBurgers: {},
      location: {
        x: 0,
        y: 0
      }
    }
  },
  methods: {
    getOrderNumber: function () {
      return Math.floor(Math.random() * 100000);
    },
    addOrder: function (event) {
      var offset = {
        x: event.currentTarget.getBoundingClientRect().left,
        y: event.currentTarget.getBoundingClientRect().top
      };
      this.location.x = event.clientX - 10 - offset.x;
      this.location.y = event.clientY - 10 - offset.y;
      socket.emit("addOrder", {
        orderId: this.getOrderNumber(),
        details: {
          x: event.clientX - 10 - offset.x,
          y: event.clientY - 10 - offset.y
        },
        orderItems: ["Beans", "Curry"]
      }
      );
    },

    setLocation(event) {
      var offset = {
        x: event.currentTarget.getBoundingClientRect().left,
        y: event.currentTarget.getBoundingClientRect().top
      };
      this.location.x = event.clientX - 10 - offset.x;
      this.location.y = event.clientY - 10 - offset.y;
    },

    addToOrder: function (event) {
      this.orderedBurgers[event.name] = event.amount;
    },

    printValues() {
      console.log('Namn:', this.fn);
      console.log('E-mail:', this.em);
      console.log('Gata:', this.street);
      console.log('Husnummer:', this.houseNumber);
      console.log('Betalningsmetod:', this.paymentMethod);
      console.log('Kön', this.gender);

      socket.emit("addOrder", {
        orderId: this.getOrderNumber(),
        details: {
          x: this.location.x,
          y: this.location.y
        },
        orderItems: this.orderedBurgers,
        customerInfo: {
          name: this.fn,
          email: this.em,
          gender: this.gender,
          paymentMethod: this.paymentMethod
        }
      
      },
      );
    },
  }
}

</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Agbalumo&family=Cormorant:wght@700&display=swap');

#map {
  width: 1920px;
  height: 1078px;
  background: url("/img/polacks.jpg");
}

.mapSection {
  width: 400px;
  height: 400px;
  overflow: scroll
}

body {
  font-size: 12pt;
  font-family: "Gill Sans", sans-serif;
}

.allergen {
  font-weight: bold;
  font-size: 12pt;
}

#burger-selection {
  background-color: black;
  color: white;
}

button:hover {
  background-color: #4CAF50;
  /* Ändra till önskad bakgrundsfärg */
  color: white;
  /* Om du vill ändra textfärgen också */
  cursor: pointer;
  /* Ändrar muspekaren till en hand */
}

section {
  margin: 20px;
  /* Skapar utrymme runt varje sektion */
  border: 2px solid #800020;
  /* Skapar en streckad ram i samma färg som texten */
  padding: 20px;
  /* Lägger till avstånd mellan ramen och texten */
  margin-bottom: 10px;
  /* Skapar avstånd mellan sektionerna */
}

button {
  margin: 20px;
}

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

.header-banner {
  position: relative;
  margin: 20px;
  /* Lägger till 20px top/bottom och auto på sidorna för att centrera */
  width: calc(100% - 40px);
  height: 200px;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  color: white;
  /* Textfärg för bättre kontrast mot bakgrunden */
}

.header-image {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  /* Gör att bilden täcker hela ytan */
  z-index: -1;
  /* Lägger bilden bakom texten */
  opacity: 0.7
}

h1 {
  font-family: 'Agbalumo';
  font-size: 36pt;
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

#dots {
  position: relative;
  margin: 0;
  padding: 0;
  background-repeat: no-repeat;
  width: 1920px;
  height: 1078px;
  cursor: crosshair;
  background-image: url('/img/polacks.jpg');
}

#dots div {
  position: absolute;
  background: black;
  color: white;
  border-radius: 10px;
  width: 20px;
  height: 20px;
  text-align: center;
}
</style>

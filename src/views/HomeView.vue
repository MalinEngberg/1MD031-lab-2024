<template>
  <div>
    <header> 
      <div class="header-content">
          <img src="/img/Cosy_Restaurant.jpg">
          <h1>Väkommen till BurgerOnline</h1>
      </div>
    </header>

    <main>
      <section id="burger-section"> 
        <div id="burger-title">
          <h2>Välj vad du är sugen på idag</h2>
          <p>Här ser du vilka hamburgare du kan välja mellan</p>
        </div>

        <div class="burger-wrapper">
          <Burger 
            v-for="burger in burgers"
            v-bind:burger="burger" 
            v-bind:key="burger.name"
            v-on:burger-ordered="handleBurgerOrder"/>
        </div>
      </section>

      <section id="order-section">
        <h2>Kundinformation</h2>
        <p>Här anger du nödvändig information (dina kontaktuppgifter, leveransadress etc)</p>
        <h3>Leveransinformation</h3>
        <p>
          <label for="fullname">Fullständigt namn:</label><br>
          <input type="text" id="fullname" v-model="fullname" required ="required" placeholder="För- & Efternamn">
        </p>
        <p>
          <label for="email">E-mail adress:</label><br>
          <input type="email" id="email" v-model = "email" required="required" placeholder="E-mail adress">
        </p>
        
        <div class="map-instructions">
          <p>Klicka på kartan nedan för att ange din leveransadress</p>
          <div id="map-container">
            <div id="map" v-on:click="setLocation"></div>
              <div v-if="location.x > 0 || location.y > 0"
                  class="location-marker"
                  v-bind:style="{ left: location.x + 'px', top: location.y + 'px' }">
                T
                </div>
          </div>
        </div>
        
        <p>
          <label for="paymentMethod">Betalnings metod:</label>
          <select id="paymentMethod" v-model="paymentMethod">
            <option selected="selected">Credit Card</option>
            <option>Swish</option>
            <option>Klarna</option>
            <option>Kontant</option>
            <option>Apple Pay</option>
          </select>
        </p>
        <p>
          <label>Ange kön:</label><br>
                      
          <input type="radio" id="male" value="male" v-model="gender">
          <label for="male">Man</label><br>
                      
          <input type="radio" id="female" value="female" v-model="gender">
          <label for="female">Kvinna</label><br>

          <input type="radio" id="non-binary" value="non-binary" v-model="gender">
          <label for="non-binary">Icke-Binär</label><br>

          <input type="radio" id="no-answer" value="no-answer" v-model="gender">
          <label for="no-answer">Jag vill inte svara</label>
        </p>
      </section>
      <button type="submit" v-on:click="submitOrder"> <!--Rätt placering?-->
        <img src="/img/burger_order2.png" style="height: 30px;">
        Skicka min order
      </button>
    </main>
    <hr> 
    <section id="receipt" v-if="lastOrder">
      <h2>Orderbekräftelse</h2>
      <p>Tack för din beställning! Här är en sammanfattning av din order:</p>
      <ul>
        <li><strong>Ordernummer:</strong> {{ lastOrder.orderId }}</li>
        <li><strong>Beställda varor:</strong>{{ lastOrder.orderItems.join(', ') }}</li>
        <li><strong>Namn:</strong> {{ fullname }}</li>
        <li><strong>E-mail:</strong> {{ email }}</li>
        <li><strong>Betalningsmetod:</strong> {{ paymentMethod }}</li>
        <li><strong>Status:</strong> {{ lastOrder.status }}</li>
        </ul>
    </section>
    <footer>
      <p>&copy; 2025 BurgerOnline Co.</p>
    </footer>
  </div>
</template>

<script>
import Burger from '../components/OneBurger.vue'
import io from 'socket.io-client'
import menuData from '@/assets/menu.json'

const socket = io("localhost:3000");

function MenuItem(name, kCal, url, lactose, gluten){ 
  this.kCal = kCal;
  this.url = url;
  this.lactose = lactose;
  this.gluten = gluten;
}

export default {
  name: 'HomeView',
  components: {
    Burger
  },
  data: function () {
    return {
      burgers: menuData,
      orderedBurgers: {},
      fullname: '',
      email: '',
      paymentMethod: 'Credit Card',
      gender: 'no-answer',
      location: {x: 0, y: 0},
      lastOrder: null
    }
  },
  methods: {
    getOrderNumber: function () {
      return Math.floor(Math.random()*100000);
    },
     handleBurgerOrder: function(burgerName, amount){
      if (amount > 0){
        this.orderedBurgers[burgerName] = amount;
      } else {
        delete this.orderedBurgers[burgerName];
      }
     },  
    
    setLocation: function(event){
      var offset = {
          x: event.currentTarget.getBoundingClientRect().left,
          y: event.currentTarget.getBoundingClientRect().top};

      this.location={x: event.clientX-5 - offset.x,
                     y: event.clientY - 5 - offset.y };
      
      console.log("Location set to:", this.location);
    },
    submitOrder: function (){
      
      if (!this.fullname || !this.email || this.location.x === 0 && this.location.y === 0){
        alert("Vänligen fyll i alla nödvändiga fält innan du skickar din order.");
        return;
      }

      var orderItems=[];      
      for (var burgerName in this.orderedBurgers){
        if(this.orderedBurgers[burgerName]>0){
          orderItems.push(burgerName + " x " + this.orderedBurgers[burgerName]);
        }
      }

      socket.emit("addOrder", 
        { orderId: this.getOrderNumber(),
          details: this.location,
          orderItems: orderItems,
          customerInfo: {
            name: this.fullname,
            email: this.email,
            paymentMethod: this.paymentMethod,
            gender: this.gender
          }
        });

      this.lastOrder = {
        orderId: this.getOrderNumber(),
        orderItems: orderItems,
        customerInfo: {
          name: this.fullname,
          email: this.email,
          paymentMethod: this.paymentMethod

        },
        status: "Mottagen"
      };
    }
  }
}
</script>

<style>
  .map-instructions{
    margin-bottom: 1em;
  }
  #map-container{
    position: relative;
    width: 100%;
    height: 500px;
    overflow: scroll;
    border: 2px solid black;
    margin-bottom: 2em;
  }
  .location-marker{
    position: absolute;
    width: 15px;
    height: 15px;
    color: white;
    background-color: rgb(98, 46, 46);
    text-align: center;
    border: 2px solid black;
    border-radius: 50%;
    font-size: 9pt;
    line-height: 15px;
    font-weight: bold;
  }

  #map {
    width: 1920px;
    height: 1078px;
    background: url("/img/polacks.jpg");
    position: relative;
    cursor: pointer;
  }

  @import url('https://fonts.googleapis.com/css2?family=Agbalumo&family=Cormorant:wght@700&display=swap');
body {
    font-size: 12pt;
    font-family: "Times New Roman";
}

h1 {
    font-family: 'Agbalumo';
    font-size: 36pt;
}
main, header, footer, nav ul {
    max-width: 90rem; /*här jag ändrar storleken den svarta bakgrunden*/
    margin: 0 auto;
}

main {
    background-color: white(198, 236, 229);
}

/* nav ul li {
    display: inline-block;
    background-color: grey;
    padding: 1em;
    margin: 1em;
} */
.header-content {
    position: relative;
    height: 200px;
    overflow: hidden;
    margin: 15px;
    border: 10px solid rgba(66, 44, 44, 0.3);
    border-radius: 5px;
}

.header-content h1 {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);  
    color: white;  
    text-shadow: 2px 2px 4px black;
    margin: 0;
    width: auto; 
}
@media only screen and (max-width: 700px){
    h1{
        font-size: 30pt;
    }
}

@media only screen and (max-width: 590px){
    h1{
        font-size: 25pt;
    }
}


.header-content img{
    width: 100%;
    height: auto;
    display: block;
    opacity: 0.7;
}

nav ul {
    display: grid;
    grid-template-columns: repeat(auto-fill, 9.25em);
    gap: 1em;
    padding: 0;
}

nav li {
    display: block;
    background-color: grey;
    padding: 1em;
}

.Very-good {
    color: green;
}

.Master {
    color: green;
    font-weight: bold;
}
#burger-section { /*ändrar bagrund och textfärg*/
    background-color: rgb(87, 52, 52);
    color: white;
    margin: 15px;
    padding: 15px;
    border: 5px dashed; /*behöver ej lägga till färg, då används automatiskt textfärgen*/
}
#burger-title{
    text-align: center;
    color: white;
}
.burger-images{
    display: block;
    margin: auto;
    
}
.burger-h3{
    text-align: center;
}
.burger-wrapper{
    display: grid;
    grid-template-columns: repeat(3, 1fr) /*repeat(3, 1fr)*/;
    /*gap: 60px;*/
}
.burger-item{
    border: 4px ridge white;
    border-radius: 10px;
    padding: 20px;
    margin: 10px;
    background-color: rgba(255, 255, 255, 0.184);
    box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);
}
@media only screen and (max-width: 900px){
    .burger-wrapper{
        grid-template-columns: repeat(1, 1fr);
    }
}

#order-section{
    background-color: rgb(154, 88, 88);
    margin: 15px;
    padding: 15px;
    border: 5px dashed;
    color: black;
}
.ingridient-info {
    font-weight: bold;
}
#receipt {
    background-color: rgb(232, 231, 231);
    margin: 15px;
    padding: 15px;
    border: 5px solid black;
    border-radius: 5px;
}
.receipt-conent {
    background: white;
    font-size: 14pt;
}
button:hover { /*ändrar "skicka order"-knappen*/
    background-color: #27F573;
    cursor: pointer; /*ändrar musen när den befinner sig inom knapp området*/
}

button {
    margin-left: 15px;
    margin-bottom: 10px;
}
</style>
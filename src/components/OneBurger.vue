<template>
  <!--<div>
    {{ burger.name }} {{ burger.kCal }}
  </div>-->
  <div class="burger-item">
    <h3 class="burger-h3">{{burger.name}}</h3>
    <img class="burger-images" v-bind:src="burger.url" style="height: 200px;"> 
    <ul>
      <li>{{ burger.description }}</li>
      <li>{{burger.kCal}} kCal</li>
      <li v-if = "burger.gluten || burger.lactose">
        Innehåller
        <span v-if="burger.gluten" class="ingridient-info"> gluten</span>
        <span v-if="burger.gluten && burger.lactose"> och </span>
        <span v-if="burger.lactose" class="ingridient-info"> laktos</span>
    </li>
    <li>{{ burger.price }}</li>
   </ul>
   <div class="order-controls">
    <p>Lägg till antal:</p>
    <div class="amount-button">
      <button v-on:click="decreaseAmount">-</button>
      <span class="amount-ordered">{{ amountOrdered }}</span>
      <button v-on:click="increaseAmount">+</button>
    </div>
   </div>
  </div>
</template>

<script>
export default {
  name: 'OneBurger',
  props: {
    burger: Object
  },
  data: function() {
  return {
    amountOrdered: 0
    }
  },
  methods: {
    increaseAmount: function() {
      this.amountOrdered++;
      this.$emit('burger-ordered', this.burger.name, this.amountOrdered);
    },
    decreaseAmount: function() {
      if (this.amountOrdered > 0) {
        this.amountOrdered--;
        this.$emit('burger-ordered', this.burger.name, this.amountOrdered);
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .burger-item{
      /*border: 4px ridge white;
      border-radius: 3px;*/
      padding: 20px;
      margin: 10px;
      /*background-color: rgba(255, 255, 255, 0.184);*/
  }
  .burger-images{
    display: block;
    margin: auto;
  }
  .ingridient-info {
    font-weight: bold;
  }
  .burger-h3{
    text-align: center;
  }
  .order-controls{
    text-align: center;
    margin-top: 1em;
  }
  .amount-button{
    text-align: center;
    font-size: 14pt;
    font-weight: bold;
  }
  .amount-button button{
    padding: 0.2em 0.5em;
    margin: 0;
    border: 0.15em solid #332b2b;
    border-radius: 0.5em;
    font-size: 14pt;
    font-family: 'Courier New', Courier, monospace;
    font-weight: bold;
    background-color: rgb(175, 132, 128);
    position: relative;  
  }
  .amount-ordered{
    display: inline-block;
    width: 1.5em;
    text-align: center;
  }
  .amount-button button:hover{
    background-color: rgb(175, 132, 128);
    color: white;
  }
 
  
</style>
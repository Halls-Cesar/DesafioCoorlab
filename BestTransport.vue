<template>
  <div class="container">
    <div class="b-title">
     <h1 class="title">Análise de Frete</h1>
    </div>
    <div class="content">
      <div class="input-wrapper">
        <label for="weight">Peso do Frete (em Kg):</label>
        <input type="number" id="weight" v-model="weight" />
      </div>
      <div class="input-wrapper">
        <label for="destination">Selecione a cidade de Destino:</label>
        <select id="destination" v-model="destination">
          <option disabled value="">Selecione...</option>
          <option>Campinas</option>
          <option>São Paulo</option>
          <option>Limeira</option>
        </select>
      </div>
      <button class="btn" @click="analyzeFreight">Analisar</button>
      <div v-if="errorMessage" class="error-message">
        <p>{{ errorMessage }}</p>
        <button class="btn btn-ok" @click="closeErrorMessage">OK</button>
      </div>
      <div v-if="bestPriceFreight || fastestFreight" class="result-container">
        <div class="result">
          <h2>Frete mais econômico:</h2>
          <p>Transportadora: {{ bestPriceFreight.name }}</p>
          <p>Custo total: {{ bestPriceFreight.totalCost }}</p>
          <p>Tempo de entrega: {{ bestPriceFreight.leadTime }}</p>
        </div>
        <div class="result">
          <h2>Frete mais rápido:</h2>
          <p>Transportadora: {{ fastestFreight.name }}</p>
          <p>Custo total: {{ fastestFreight.totalCost }}</p>
          <p>Tempo de entrega: {{ fastestFreight.leadTime }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      weight: 0,
      destination: '',
      bestPriceFreight: null,
      fastestFreight: null,
      errorMessage: '',
    };
  },
  methods: {
    analyzeFreight() {
      if (this.weight === 0 || this.destination === '') {
        this.errorMessage = 'Preencha os Dados';
        return;
      }

      axios.get('http://localhost:3000/transport')
        .then(response => {
          const data = response.data;
          let bestPrice = Number.MAX_SAFE_INTEGER;
          let fastestTime = Number.MAX_SAFE_INTEGER;
          let bestPriceFreight = null;
          let fastestFreight = null;

          data.forEach(freight => {
            const cost = this.weight <= 100 ? freight.cost_transport_light : freight.cost_transport_heavy;
            const totalCost = parseFloat(cost.replace('R$', '')) * this.weight;
            const leadTime = freight.lead_time;

            if (totalCost < bestPrice) {
              bestPrice = totalCost;
              bestPriceFreight = {
                name: freight.name,
                totalCost: `R$ ${totalCost.toFixed(2)}`,
                leadTime,
              };
            }

            if (parseInt(leadTime) < fastestTime) {
              fastestTime = parseInt(leadTime);
              fastestFreight = {
                name: freight.name,
                totalCost: `R$ ${totalCost.toFixed(2)}`,
                leadTime,
              };
            }
          });

          this.bestPriceFreight = bestPriceFreight;
          this.fastestFreight = fastestFreight;
          this.errorMessage = '';
        })
        .catch(error => {
          console.error(error);
        });
    },
    closeErrorMessage() {
      this.errorMessage = '';
    },
  },
};
</script>

<style>
body {
  background-color: #87CEFA;
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
}

.b-title {
  background-color: #4169E1;
  width: 500px;
  height: 50px;
  margin: 10px;
  border-radius: 12px;
  box-shadow: 0 3px 5px 0 #00008B;
}

h1 {
  text-align: center;
}

.title {
  color: whitesmoke;
  font-size: 24px;
  margin-bottom: 7px;
  margin-top: 7px;
}

.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
}

.content {
  background-color: #fff;
  border-radius: 7px;
  padding: 10%;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
}

.input-wrapper {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

label {
  margin-right: 10px;
}

input[type="number"],
select {
  padding: 5px;
  border-radius: 3px;
  border: 1px solid #ccc;
}

.btn {
  background-color: #000000;
  color: #000000;
  border: none;
  padding: 8px 16px;
  border-radius: 3px;
  cursor: pointer;
}

.error-message {
  background-color: #ff9999;
  padding: 10px;
  margin-top: 10px;
  border-radius: 5px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.btn-ok {
  background-color: #ff5555;
  color: #fff;
  border: none;
  padding: 5px 10px;
  border-radius: 3px;
  cursor: pointer;
}

.result-container {
  margin-top: 20px;
  display: flex;
  justify-content: space-between;
}

.result {
  padding: 10px;
  background-color: #f5f5f5;
  border-radius: 5px;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.2);
}

.result h2 {
  margin-top: 0;
  margin-bottom: 10px;
  font-size: 18px;
}

.result p {
  margin: 5px 0;
}
</style>

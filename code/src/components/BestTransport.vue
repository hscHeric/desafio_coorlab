<template>
  <div class="title">
    <b-navbar toggleable="lg" type="dark" variant="info">
      <b-navbar-brand class="ml-2"></b-navbar-brand>
    </b-navbar>
    <div class="m-card container mt-3">
      <div class="m-card-header">
        <h4><b-icon-truck></b-icon-truck> {{ appName }}</h4>
      </div>
      <div class="m-card-body container">
        <div class="m-gray-card">
          <div class="m-gray-card-header">
            <h3><b-icon-map></b-icon-map> Insira o destino e o peso</h3>
          </div>
          <div class="m-gray-card-body">
            <div class="form-group ">
              <label for="FormControlSelect">Destino</label>
              <b-form-select class="form-control" id="FormControlSelect" v-model="selectedCity">
                <template #first>
                  <b-form-select-option :value="null" disabled> Selecione o destino </b-form-select-option>
                </template>
                <option v-for="city in cities" :key="city" :value="city">{{ city }}</option>
              </b-form-select>
              <label for="weight-form">Peso</label>
              <b-form-input type="string" id="weight-form" v-model="weight" placeholder="Peso da carga em kg"
                class="mb-3 weight_form"></b-form-input>
              <div class="text-center m-center">
                <button class="btn btn-primary m-button" @click="findBestTransporters">Analisar</button>
              </div>
            </div>
          </div>
        </div>
        <div class="m-card-no-color">
          <div class="query-info" v-if="filteredTransporters">
            <div>
              <h3>
                Estas são as melhores alternativas de frete que encontramos para você.
              </h3>
              <div class="card-transporter">
                <div class="pt-01">
                  <div class="m-icon">
                    <b-icon-cash-coin></b-icon-cash-coin>
                  </div>
                  <div class="m-info">
                    <h5>Frete com o menor valor</h5>
                    <p>Transportadora: {{ filteredTransporters.transportersWithShortestLeadTime.transporter.name }}</p>
                    <p>Tempo estimado: {{ filteredTransporters.transportersWithShortestLeadTime.transporter.lead_time }}
                    </p>
                  </div>
                </div>
                <div class="pt-02">
                  <h5>Preço</h5>
                  <p>{{ "R$" + filteredTransporters.transportersWithShortestLeadTime.price.toFixed(2).replace(".", ",") }}
                  </p>
                </div>
              </div>
              <div class="card-transporter">
                <div class="pt-01">
                  <div class="m-icon">
                    <b-icon-alarm></b-icon-alarm>
                  </div>
                  <div class="m-info">
                    <h5>Frete mais rápido</h5>
                    <p>Transportadora: {{ filteredTransporters.transportersWithLowestPrice.transporter.name }}</p>
                    <p>Tempo estimado: {{ filteredTransporters.transportersWithLowestPrice.transporter.lead_time }}</p>
                  </div>
                </div>
                <div class="pt-02">
                  <h5>Preço</h5>
                  <p>{{ "R$" + filteredTransporters.transportersWithLowestPrice.price.toFixed(2).replace(".", ",") }}</p>
                </div>
              </div>
            </div>
            <div class="m-align-left">
              <button class="btn btn-primary m-button" @click="ClearFilteredTransporters">Limpar</button>
            </div>
          </div>
          <div class="query-null-message text-center" v-if="!filteredTransporters">
            <h3>Nenhum transportador selecionado</h3>
          </div>
        </div>
      </div>
    </div>
    <div>
      <b-modal v-if="modalVisible" v-model="modalVisible" title="Insira os valores para realizar a análise" hide-footer
        hide-close-button>
        <p>{{ modalMessage }}</p>
        <div class="text-center">
          <button class="btn btn-primary m-button" @click="hideModal">Fechar</button>
        </div>
      </b-modal>
    </div>
  </div>
</template>


<style scoped>
* {
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
}

.m-button {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 0;
  height: 30px;
  width: 160px;
  background: #91B5C0 !important;
  align-self: center;
  border: #91B5C0;
}

.card-transporter {
  margin: 14px 0px;
  align-items: right;
  display: flex;
  width: 100% !important;
  height: 100px;
  border-radius: 5px;
  justify-content: space-between;
  overflow: hidden;
}

.card-transporter .pt-01 {
  display: flex;
  align-items: center;
  border-radius: 5px;
  width: 70%;
  background-color: #F3F3EF;
}

.card-transporter .m-icon {
  font-size: xx-large;
  margin-right: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #92B5C6;
  width: 90px;
  height: 100%;
}

.m-expand {
  height: 100%;
}

.card-transporter .pt-01 p {
  margin: 0;
}

.card-transporter .pt-02 p {
  margin: 0;
}

.m-align-left {
  display: flex;
  justify-content: flex-end;
}

.card-transporter .pt-02 {
  display: flex;
  flex-direction: column;
  justify-content: center;
  border-radius: 5px;
  padding: 20px;
  width: 28%;
  height: 100%;
  background-color: #F3F3EF;
}

.title .navbar {
  background-color: #2e6782 !important;
}

.query-null-message {
  margin-left: 150px;
  margin-right: 150px;
}

.m-card-no-color {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
}

.m-card {
  border-top-right-radius: 10px;
  border-top-left-radius: 10px;
  border-bottom-left-radius: 5px;
  border-bottom-right-radius: 5px;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  padding: 0;
  margin-top: 35px !important;
  box-shadow: -2px 3px 27px -1px rgba(0, 0, 0, 0.3);
  -webkit-box-shadow: -2px 3px 27px -1px rgba(0, 0, 0, 0.3);
  -moz-box-shadow: -2px 3px 27px -1px rgba(0, 0, 0, 0.3);
}

.m-card .m-card-header {
  background-color: #92B5C6;
  height: 55px;
  padding: 0px 30px;
  font-size: large;
  font-weight: bold;
  display: flex;
  align-items: center;
}

.m-card .m-card-header h4 {
  margin: 0px !important;
}

.m-card .m-card-body {
  padding: 15px;
  display: flex;
  height: 70dvh;
}

.m-gray-card-header {
  text-align: center;
}

.m-center {
  display: flex;
  justify-content: center;
  align-items: center;
}

.query-info {
  display: flex;
  justify-content: space-between;
  flex-direction: column;
  padding: 20px;
  width: 100%;
  height: 100%;
}

.title .navbar-brand {
  margin-left: 20px;
}

.form-group {
  margin-top: 0px !important;
}

.m-gray-card {
  display: flex;
  flex-direction: column;
  justify-content: center;
  justify-content: center;
  background-color: #F0F0F0;
  border-radius: 8px;
  min-width: 400px;
  height: 100%;
}

.m-gray-card-body {
  padding: 20px;
}

.form-group {
  margin-top: 80px;
  height: 100%;
}
</style>

<script>
import {
  BNavbar,
  BNavbarBrand
} from 'bootstrap-vue'

export default {
  components: {
    BNavbar,
    BNavbarBrand
  },
  data() {
    const appName = ''
    const transport = []
    const cities = []

    return {
      appName,
      transport,
      cities,
      selectedCity: null,
      weight: null,
      filteredTransporters: undefined,
      modalVisible: false,
    }
  },
  created() {
    // Implemente aqui o GET dos dados da API REST
    // para que isso ocorra na inicialização da pagina
    this.appName = 'Melhor Frete';
    fetch('http://localhost:3000/transport')
      .then(response => response.json())
      .then(data => {
        this.transport = data;
        this.cities = this.GetAllCities(data);
      }).catch(error => {
        console.error('Erro ao obter os dados da API:', error);
      });

  },
  methods: {
    hideModal() {
      this.modalVisible = false;
    },

    //Executa as funções para encontrar os melhores transportadores em relação ao tempo de entrega e ao preço e armazena os resultados em um array
    findBestTransporters() {
      const city = this.selectedCity;
      const weight = parseFloat(this.weight.replace(',', '.').replace('kg', '').trim());

      if (city == null && weight == null) {
        this.modalMessage = 'A cidade e o peso não foram selecionados.';
        this.modalVisible = true;
        return;
      }

      if (city == null) {
        this.modalMessage = 'A cidade não foi selecionada.';
        this.modalVisible = true;
        return;
      }

      if (weight == null) {
        this.modalMessage = 'O peso não foi colocado.';
        this.modalVisible = true;
        return;
      }

      if (isNaN(weight) || weight <= 0) {
        this.modalMessage = 'O valor do peso não é um número válido.';
        this.modalVisible = true;
        return;
      }
      this.selectedCity = null;
      this.weight = null;
      const transportersWithShortestLeadTime = this.findTransporterWithShortestLeadTime(city, weight);
      const transportersWithLowestPrice = this.findTransporterWithLowestPrice(city, weight);

      this.filteredTransporters = {
        transportersWithShortestLeadTime,
        transportersWithLowestPrice
      };
    },


    OpenModal() {
      this.modalVisible = true;
    },

    CloseModal() {
      this.modalVisible = false;
    },

    //Limpa o array de transportadores filtrados
    ClearFilteredTransporters() {
      this.filteredTransporters = undefined;
    },

    //Retorna o transportador com o menor tempo de entrega e, se houver um empate no tempo de entrega, seleciona o que tiver o menor preço.
    findTransporterWithShortestLeadTime(city, weight) {
      let shortestLeadTime = Infinity;
      let shortestLeadTimeTransporter = null;
      let lowestPrice = Infinity;
      let lowestPriceTransporter = null;

      const TransportersFilteredByCity = this.GetTransportersByCity(city);

      TransportersFilteredByCity.forEach(transporter => {
        const leadTime = parseInt(transporter.lead_time.replace('h', '').trim());
        const convertedPrices = this.ConvertPriceToFloat(transporter);
        const price = weight <= 100 ? convertedPrices.costTransportLight * weight : convertedPrices.costTransportHeavy * weight;

        if (leadTime < shortestLeadTime || (leadTime === shortestLeadTime && price < lowestPrice)) {
          shortestLeadTime = leadTime;
          shortestLeadTimeTransporter = transporter;
          lowestPrice = price;
          lowestPriceTransporter = transporter;
        }
      });

      if (shortestLeadTimeTransporter === null || lowestPriceTransporter === null) {
        return null;
      }
      const convertedPrices = this.ConvertPriceToFloat(shortestLeadTimeTransporter);
      if (weight <= 100) {
        lowestPrice = convertedPrices.costTransportLight * weight;
      } else {
        lowestPrice = convertedPrices.costTransportHeavy * weight;
      }

      return {
        transporter: shortestLeadTimeTransporter,
        price: lowestPrice
      };
    },

    findTransporterWithLowestPrice(city, weight) {
      let lowestPrice = Infinity;
      let lowestPriceTransporter = null;
      let lowestLeadTime = Infinity;
      let lowestLeadTimeTransporter = null;

      const TransportersFilteredByCity = this.GetTransportersByCity(city);


      TransportersFilteredByCity.forEach(transporter => {
        const convertedPrices = this.ConvertPriceToFloat(transporter);
        let price;
        if (weight <= 100) {
          price = convertedPrices.costTransportLight * weight;
        } else {
          price = convertedPrices.costTransportHeavy * weight;
        }

        const leadTime = parseInt(transporter.lead_time.replace('h', '').trim());

        if (price < lowestPrice || (price === lowestPrice && leadTime < lowestLeadTime)) {
          lowestPrice = price;
          lowestPriceTransporter = transporter;
          lowestLeadTime = leadTime;
          lowestLeadTimeTransporter = transporter;
        }
      });

      if (lowestPriceTransporter === null || lowestLeadTimeTransporter === null) {
        return null;
      }
      const convertedPrices = this.ConvertPriceToFloat(lowestPriceTransporter);
      if (weight <= 100) {
        lowestPrice = convertedPrices.costTransportLight * weight;
      } else {
        lowestPrice = convertedPrices.costTransportHeavy * weight;
      }

      return {
        transporter: lowestPriceTransporter,
        price: lowestPrice
      };
    },

    //Recebe uma transportadora e converte os preços para float e retorna um array com os preços
    ConvertPriceToFloat(transportItem) {
      const costTransportLight = parseFloat(transportItem.cost_transport_light.replace('R$', '').trim());
      const costTransportHeavy = parseFloat(transportItem.cost_transport_heavy.replace('R$', '').trim());

      return {
        costTransportLight,
        costTransportHeavy
      }
    },

    //Retorna um array com todas as cidades para serem usadas no select
    GetAllCities(data) {
      const cities = [];
      data.forEach((Transporter) => {
        if (!cities.includes(Transporter.city)) {
          cities.push(Transporter.city);
        }
      });
      return cities;
    },

    //Retorna um array com os transportadores filtrados pela cidade armazenada em selectedCity
    GetTransportersBySelectedCity() {
      const city = this.selectedCity;
      this.filteredTransporters = this.getTransportersByCity(city);
    },
    //Retorna um array com os transportadores filtrados pela cidade selecionada
    GetTransportersByCity(city) {
      return this.transport.filter(item => item.city === city)
    },
    methodFoo() {
      console.log(this.appName)
    },
  },
}
</script>
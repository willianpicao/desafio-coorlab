<template>

<div class="title">
  <b-navbar toggleable="lg" type="dark" variant="info">
    <b-navbar-brand class="ml-2">
      <b>{{ appName }}</b>
    </b-navbar-brand>
  </b-navbar>
  <div class="main-container">
    <div class="form-container">
      <form @submit.prevent="submitForm">
        <!-- botao para selecionar cidades -->
        <select class="form-control" v-model="citySelected">
          <option value="" disabled>Selecione o destino</option>
          <option v-for="item in citys" :value="item" :key="item">{{ item }}</option>
        </select>
        <!-- Input do weightSelected -->
        <input v-model="weightSelected" placeholder="Informe o peso em kg" type="text" />

        <button type="submit">Selecionar</button>
      </form>
    </div>

    <div class="result-container" v-if="showResult">
      <div class="rectangle"></div>
      <div class="rectangle"></div>
      <button @click="clearResult">Limpar</button>
    </div>

    <div class="modal" v-if="showModal">
      <div class="modal-content">
        <h3>Preencha todos os campos</h3>
        <p>Por favor, preencha todos os campos antes de continuar.</p>
        <button @click="closeModal">Fechar</button>
      </div>
    </div>
  </div>
  <div class="modal" v-if="showModal">
  <div class="modal-content">
    <h3>Preencha todos os campos</h3>
    <p>Por favor, preencha todos os campos antes de continuar.</p>
    <button @click="closeModal">Fechar</button>
  </div>
</div>
</div>



</template>

<script>
import axios from 'axios'
import {
  BNavbar,
  BNavbarBrand,
} from 'bootstrap-vue'

export default {
  components: {
    BNavbar,
    BNavbarBrand,
  },
  data() {
    const appName = ''
    const citys={}
    const freteCity=[]
    const lowestPrice=Infinity
    const shorterTime=Infinity
    const nameLesserPrice=''
    const nameLesserTime=''
    const showModal=false
    const showResult=false
    return {
      showResult,
      showModal,
      nameLesserTime,
      nameLesserPrice,
      lowestPrice,
      shorterTime,
      freteCity,
      appName,
      citys,
      citySelected:'',
      weightSelected:''
    }
  },
  async created() {
    const { data } = await axios.get('http://localhost:3000/transport');
    let citys = {};
    data.map((item) => {
      citys[item.city] = 1;
    })
    this.citys = Object.keys(citys);

    this.appName = 'Melhor Frete'
    
  },
  methods: {
    // Implemente aqui os metodos utilizados na pagina
    methodFoo() {
      console.log(this.appName)
    },
    async submitForm ( ){
      if (!this.citySelected || !this.weightSelected) {
        this.showModal = true; // Exibe a tela/modal caso algum campo não tenha sido preenchido
        return;
      }
      // Pega os valores das variaveis do formulario 
      // Sendo eles valores dos campos do SELECT da Cidade
      // E o Peso do Frete
      const city= this.citySelected
      const weight= parseInt(this.weightSelected)

      console.log("cidade:"+city)
      console.log("tipo da varial weigth "+ typeof(weight));
      
      //Pega todos objetos da API
      const { data } = await axios.get('http://localhost:3000/transport');
      let freteCity = [];
      //Filtra os objetos da API baseado na escolha de cidade do Usuario
      data.map((item) => {
        if (item.city == city){
          freteCity.push(item)
        }
      })
      this.freteCity = freteCity
      console.log(freteCity);

      //por peso verificando se > 100 ou <
      if(weight > 100){
        freteCity.forEach(objeto => {
          const value = parseFloat(objeto.cost_transport_heavy.substring(3));

          if (value < this.lowestPrice) {
            this.lowestPrice = value;
            this.nameLesserPrice = objeto.name;
          }
        });

      }else if ( (weight > 0) && (weight <= 100) ){
        freteCity.forEach(objeto => {
          const value = parseFloat(objeto.cost_transport_light.substring(3));

          if (value < this.lowestPrice) {
            this.lowestPrice = value;
            this.nameLesserPrice = objeto.name;
          }
        });
      }

      //Menor tempo
      freteCity.forEach(objeto => {
          const time = parseInt(objeto.lead_time.slice(0, -1));

          if (time < this.shorterTime) {
            this.shorterTime = time;
            this.nameLesserTime = objeto.name;
          }
        });

      //veriicar nome da compania com menor preço e preço
      console.log("Menor preco= "+this.lowestPrice);
      console.log("Nome tranportadora de menor preço= "+this.nameLesserPrice);

      console.log("menor tempo= "+ this.shorterTime);
      console.log("nome compania menor tempo= "+ this.nameLesserTime);
      this.showResult=true
    },
    closeModal(){
      this.showModal=false
    },
    clearResult(){
      this.showResult=false
      this.clearForm()
    },
    clearForm() {
      this.citySelected = ''; // Limpa o valor do campo citySelected
      this.weightSelected = ''; // Limpa o valor do campo weightSelected
    }
  }
}
</script>

<style scoped>
.title .navbar {
  background-color: #00aca6 !important;
}

.title .navbar-brand {
  margin-left: 20px;
}



.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  background-color: #fff;
  padding: 20px;
  border-radius: 5px;
  text-align: center;
  width: 50%; /* Largura do modal */
  max-width: 500px; /* Largura máxima do modal */
  margin: 0 25%; /* Espaço de 25% de padding em cada lado */
}
.main-container {
  position: absolute;
  top: 15%;
  left: 10%;
  width: 80%;
  height: 70%;
  display: flex;
}

.form-container {
  width: 50%;
  height: 100%;
  padding: 20px;
  background-color: #f2f2f2;
}

.form-container form {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.result-container {
  width: 50%;
  height: 100%;
  padding: 20px;
  background-color: #e6e6e6;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 10px;
}

.rectangle {
  width: 100%;
  height: 100px;
  background-color: #ccc;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  background-color: #fff;
  padding: 20px;
  border-radius: 5px;
  text-align: center;
}
</style>

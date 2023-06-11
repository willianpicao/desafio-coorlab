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
        <select class="form-control" v-model="cidadeSelecionada">
          <option value="" disabled>Selecione o destino</option>
          <option v-for="item in cidades" :value="item" :key="item">{{ item }}</option>
        </select>
        <!-- Input do pesoInformado -->
        <input v-model="pesoInformado" placeholder="Informe o peso em kg" type="text" />

        <button type="submit">Selecionar</button>
      </form>
    </div>

    <div class="result-container" v-if="mostrarResultado">
      <div class="rectangle">
        <h1>Frete com menor valor</h1>
        <p>Transportadora: {{ this.companiaMenorPreco }}</p>
        <p>Tempo estimado: {{ this.tempoFreteMenorPreco }}</p>
        <p>Preço: {{ this.custoTotalFreteMenorPreco }}</p>
      </div>
      <div class="rectangle">
        <h1>Frete mais rápido</h1>
        <p>Transportadora: {{ this.companiaMenorTempo }}</p>
        <p>Tempo estimado: {{ this.tempoFreteMenortempo }}</p>
        <p>Preço: {{ this.custoTotalFreteMenorTempo }}</p>
      </div>
      <button @click="clearResult">Limpar</button>
    </div>

    <div class="modal" v-if="mostrarModal">
      <div class="modal-content">
        <h3>Preencha todos os campos</h3>
        <p>Por favor, preencha todos os campos antes de continuar.</p>
        <button @click="closeModal">Fechar</button>
      </div>
    </div>
  </div>
  <div class="modal" v-if="mostrarModal">
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
    const cidades={}
    const mostrarModal=false
    const mostrarResultado=false
    const companiaMenorPreco=''
    const companiaMenorTempo=''
    const tempoFreteMenorPreco=''
    const tempoFreteMenortempo=''
    const custoTotalFreteMenorPreco=0.0
    const custoTotalFreteMenorTempo=0.0
    return {
      companiaMenorPreco,
      companiaMenorTempo,
      tempoFreteMenorPreco,
      tempoFreteMenortempo,
      custoTotalFreteMenorPreco,
      custoTotalFreteMenorTempo,
      mostrarResultado,
      mostrarModal,
      appName,
      cidades,
      cidadeSelecionada:'',
      pesoInformado:''
    }
  },
  async created() {
    const { data } = await axios.get('http://localhost:3000/transport');
    let cidades = {};
    data.map((item) => {
      cidades[item.city] = 1;
    })
    this.cidades = Object.keys(cidades);

    this.appName = 'Melhor Frete'
    
  },
  methods: {
    // Implemente aqui os metodos utilizados na pagina
    methodFoo() {
      console.log(this.appName)
    },
    async submitForm ( ){
      if (!this.cidadeSelecionada || !this.pesoInformado) {
        this.mostrarModal = true; // Exibe a tela/modal caso algum campo não tenha sido preenchido
        return;
      }
      const { data } = await axios.get('http://localhost:3000/transport');
      console.log(data)
      const objetosFiltrados = data.filter((item) => item.city === this.cidadeSelecionada);
      console.log(objetosFiltrados);

      // Encontrar o objeto com menor lead_time
      const objetoMenorLeadTime = objetosFiltrados.reduce((minObjeto, objeto) => {
        const leadTimeMinObjeto = parseInt(minObjeto.lead_time);
        const leadTimeObjeto = parseInt(objeto.lead_time);

        if (leadTimeObjeto < leadTimeMinObjeto) {
          return objeto; // O objeto atual tem um lead_time menor, então o escolhemos como o novo objeto com menor lead_time
        } else if (leadTimeObjeto === leadTimeMinObjeto) {
          // O lead_time é igual, vamos comparar o custo

          // Extrair os custos dos objetos
          const custoMinObjeto = parseInt(minObjeto.pesoInformado) <= 100 ? parseFloat(minObjeto.cost_transport_light.replace('R$', '').trim()) : parseFloat(minObjeto.cost_transport_heavy.replace('R$', '').trim());
          const custoObjeto = parseInt(objeto.pesoInformado) <= 100 ? parseFloat(objeto.cost_transport_light.replace('R$', '').trim()) : parseFloat(objeto.cost_transport_heavy.replace('R$', '').trim());

          if (custoObjeto < custoMinObjeto) {
            return objeto; // O objeto atual tem um custo menor, então o escolhemos como o novo objeto com menor lead_time e menor custo
          }
        }

        return minObjeto; // Mantemos o objeto com menor lead_time e menor custo
      });

      console.log(objetoMenorLeadTime); //objeto de menor tempo...

      const objetoMenorValor = objetosFiltrados.reduce((minObjeto, objeto) => {
        // Converte os valores do custo em números para facilitar a comparação
        const custoMinObjeto = this.pesoInformado <= 100 ? parseFloat(minObjeto.cost_transport_light.substr(3)) : parseFloat(minObjeto.cost_transport_heavy.substr(3));
        const custoObjeto = this.pesoInformado <= 100 ? parseFloat(objeto.cost_transport_light.substr(3)) : parseFloat(objeto.cost_transport_heavy.substr(3));

        // Retorna o objeto com menor valor
        return custoObjeto < custoMinObjeto ? objeto : minObjeto;
      });

      console.log(objetoMenorValor);

      let custoTotalObjMenorPreço=0
      let custoTotalObjMenorLead=0
      let weight= parseInt(this.pesoInformado)

      console.log("peso:"+ weight);
      //Custos
      if( (weight > 0) && (weight <= 100) ){
        custoTotalObjMenorLead = (weight * parseFloat(objetoMenorLeadTime.cost_transport_light.substr(3))).toFixed(2)
        custoTotalObjMenorPreço = (weight * parseFloat(objetoMenorValor.cost_transport_light.substr(3))).toFixed(2)
      }else if ( weight > 100 ){
        custoTotalObjMenorLead = (weight * parseFloat(objetoMenorLeadTime.cost_transport_heavy.substr(3))).toFixed(2)
        custoTotalObjMenorPreço = (weight * parseFloat(objetoMenorValor.cost_transport_heavy.substr(3))).toFixed(2)
      }
      console.log("Custo total para menor valor: "+custoTotalObjMenorPreço);
      console.log("Custo total para frete mais rápido: "+custoTotalObjMenorLead);

      this.companiaMenorPreco= objetoMenorValor.name
      this.tempoFreteMenorPreco= objetoMenorValor.lead_time
      this.custoTotalFreteMenorPreco= custoTotalObjMenorPreço


      this.companiaMenorTempo= objetoMenorLeadTime.name
      this.tempoFreteMenortempo=objetoMenorLeadTime.lead_time
      this.custoTotalFreteMenorTempo=custoTotalObjMenorLead

      this.mostrarResultado=true
    },
    closeModal(){
      this.mostrarModal=false
    },
    clearResult(){
      this.mostrarResultado=false
      this.clearForm()
    },
    clearForm() {
      this.cidadeSelecionada = ''; // Limpa o valor do campo cidadeSelecionada
      this.pesoInformado = ''; // Limpa o valor do campo pesoInformado
    },
     inputClear(){
      this.cidadeSelecionada = '';
      this.pesoInformado = '';
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

<template>

<div class="title">
  <b-navbar toggleable="lg" type="dark" variant="info">
    <b-navbar-brand class="ml-2">
      <b>{{ appName }}</b>
    </b-navbar-brand>
  </b-navbar>
  <div class="main-container">
    <div class="form-container">
      <img src="../assets/logo.png">
      <h1>Insira o destino e o peso</h1>
      <form @submit.prevent="submitForm">
        <!-- botao para selecionar cidades -->
        <p>Destino</p>
        <select class="form-control" v-model="cidadeSelecionada">
          <option value="" disabled>Selecione o destino</option>
          <option v-for="item in cidades" :value="item" :key="item">{{ item }}</option>
        </select>
        <!-- Input do pesoInformado -->
        <p>Peso</p>
        <input v-model="pesoInformado" placeholder="Informe o peso em kg" type="text" />

        <button type="submit">Selecionar</button>
      </form>
    </div>

    <div class="result-container" v-if="mostrarResultado">
      <p class="pFrase">Estas são as melhores alternativas de frete que encontramos para você</p>
      <div class="rectangle">

        <div class="conteudoRect">
          <p class="titulo">Frete com menor valor</p>
          <p class="pInf">Transportadora: {{ this.companiaMenorPreco }}</p>
          <p class="pInf">Tempo estimado: {{ this.tempoFreteMenorPreco }}</p>
          <div class="preco-container">
            <p class="preco">R$ {{ this.custoTotalFreteMenorPreco }}</p>
          </div>
        </div>
        
      </div>
      <div class="rectangle">
        <div class="conteudoRect">
          <p class="titulo">Frete mais rápido</p>
          <p class="pInf">Transportadora: {{ this.companiaMenorTempo }}</p>
          <p class="pInf">Tempo estimado: {{ this.tempoFreteMenortempo }}</p>
          <div class="preco-container">
            <p class="preco">R$ {{ this.custoTotalFreteMenorTempo }}</p>
          </div>
        </div>
        
      </div>
      <button class="botaoLimpar" @click="limparResultado">Limpar</button>
    </div>

    <div class="modal" v-if="mostrarModal">
      <div class="modal-content">
        <h3>Preencha todos os campos</h3>
        <p>Por favor, preencha todos os campos antes de continuar.</p>
        <button @click="fecharModal">Fechar</button>
      </div>
    </div>
  </div>

  <div class="modal" v-if="mostrarModalInputValido">
    <div class="modal-content">
      <h3>Informe Valores Validos</h3>
      <p>Por favor, preencha o campo de peso apenas com números.</p>
      <button @click="fecharModalInputValido">Fechar</button>
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
    const mostrarModalInputValido=false
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
      mostrarModalInputValido,
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

      const weightRegex = /^[0-9]+(\.[0-9]{1,2})?$/;
      if (!weightRegex.test(this.pesoInformado)) {
        // O valor informado não é um número válido
        this.isPesoValido = false; // Define o campo de peso como inválido
        this.mostrarModalInputValido = true; // Exibe o modal de input inválido
        return;
      }


      const { data } = await axios.get('http://localhost:3000/transport');

      const objetosFiltrados = data.filter((item) => item.city === this.cidadeSelecionada);

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

      const objetoMenorValor = objetosFiltrados.reduce((minObjeto, objeto) => {
        // Converte os valores do custo em números para facilitar a comparação
        const custoMinObjeto = this.pesoInformado <= 100 ? parseFloat(minObjeto.cost_transport_light.substr(3)) : parseFloat(minObjeto.cost_transport_heavy.substr(3));
        const custoObjeto = this.pesoInformado <= 100 ? parseFloat(objeto.cost_transport_light.substr(3)) : parseFloat(objeto.cost_transport_heavy.substr(3));

        // Retorna o objeto com menor valor
        return custoObjeto < custoMinObjeto ? objeto : minObjeto;
      });

      let custoTotalObjMenorPreço=0
      let custoTotalObjMenorLead=0
      let peso= parseInt(this.pesoInformado)

      //Custos
      if( (peso > 0) && (peso <= 100) ){
        custoTotalObjMenorLead = (peso * parseFloat(objetoMenorLeadTime.cost_transport_light.substr(3))).toFixed(2)
        custoTotalObjMenorPreço = (peso * parseFloat(objetoMenorValor.cost_transport_light.substr(3))).toFixed(2)
      }else if ( peso > 100 ){
        custoTotalObjMenorLead = (peso * parseFloat(objetoMenorLeadTime.cost_transport_heavy.substr(3))).toFixed(2)
        custoTotalObjMenorPreço = (peso * parseFloat(objetoMenorValor.cost_transport_heavy.substr(3))).toFixed(2)
      }

      this.companiaMenorPreco= objetoMenorValor.name
      this.tempoFreteMenorPreco= objetoMenorValor.lead_time
      this.custoTotalFreteMenorPreco= custoTotalObjMenorPreço

      this.companiaMenorTempo= objetoMenorLeadTime.name
      this.tempoFreteMenortempo=objetoMenorLeadTime.lead_time
      this.custoTotalFreteMenorTempo=custoTotalObjMenorLead

      this.mostrarResultado=true
    },
    fecharModal(){
      this.mostrarModal=false
    },
    fecharModalInputValido(){
      this.mostrarModalInputValido=false
    },
    limparResultado(){
      this.mostrarResultado=false
      this.limparForm()
    },
    limparForm() {
      this.cidadeSelecionada = ''; // Limpa o valor do campo cidadeSelecionada
      this.pesoInformado = ''; // Limpa o valor do campo pesoInformado
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
  width: 50%;
  max-width: 500px;
  margin: 0 25%;
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
  text-align: center;
  width: 50%;
  height: 100%;
  padding: 20px;
  background-color: #f2f2f2;
}

.form-container h1 {
  font-family: 'Impact', fantasy;
  font-size: 28px;
  font-weight: bold;
  color: #333;
  margin-bottom: 20px;
}

.form-container p {
  padding-left: 1%;
  text-align: left;
  font-family: 'Arial', sans-serif;
  font-size: 16px;
  color: #666;
  margin-bottom: 8px;
  
}

.form-container input,
.form-container select {
  font-family: 'Arial', sans-serif;
  font-size: 14px;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.form-container button {
  font-family: 'Arial', sans-serif;
  font-size: 16px;
  padding: 10px 20px;
  background-color: #333;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.form-container button:hover {
  background-color: #555;
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

.pFrase{
  font-family: 'Impact', fantasy;
  font-size: 16px;
  font-weight: bold;
  color: #333;
  margin-bottom: 20px;
  border-radius: 5px;
  background-color: #333;
  width: 100%;
  text-align: center;
  padding: 5px;
  color: #fff;
}

.rectangle {
  width: 100%;
  max-height: 250px;
  background-color: #ccc;
  overflow: hidden;
  padding: 10px;
  box-sizing: border-box;
}

.rectangle h1, .rectangle p {
  margin: 0;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
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

.titulo{
  border-radius: 5px;
  background-color: #00aca6;
  width: 100%;
  text-align: center;
  padding: 5px;
  text-shadow:
    -1px -1px 1px rgba(0, 0, 0, 0.8),
    1px 1px 1px rgba(0, 0, 0, 0.8);
  color: #fff;
  font-size: 24px;
}
.preco-container {
  border-radius: 5px;
  background-color: green;
}
.pInf{
  color: #fff;
  text-shadow:
    -1px -1px 1px rgba(0, 0, 0, 0.8),
    1px 1px 1px rgba(0, 0, 0, 0.8);
}
.preco {
  text-align: center;
  padding: 5px;
  color: #fff;
  text-shadow:
    -1px -1px 1px rgba(0, 0, 0, 0.8),
    1px 1px 1px rgba(0, 0, 0, 0.8);
}

.botaoLimpar{
  font-family: 'Arial', sans-serif;
  font-size: 16px;
  padding: 10px 20px;
  background-color: #333;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.botaoLimpar:hover {
  background-color: #555;
}

.form-container img{
  width: 75px;
  height: 75px;
}


</style>

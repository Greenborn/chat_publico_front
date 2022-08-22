<template>
  <div class="row">
    <div class="col mensaje-cont">
      <div class="row" v-for="(msg) in mensajes" :key="msg">
        {{msg.texto}}
      </div>
    </div>
  </div>

  <div class="row">
    <div class="col">
      <b-form-input
            id="input-msg"
            v-model="mensaje.texto"
            type="text" placeholder="Mensaje"
            required></b-form-input>
    </div>

    <div class="col">
      <b-button variant="success" @click="enviarMensaje()">Enviar</b-button>
    </div>
  </div>
</template>

<script setup>
  import { ref, onMounted } from 'vue';

  const mensajes  = ref([])
  const conexion  = ref({})

  const mensaje = ref({
    texto: '',
    autor: 5567
  })

  function enviarMensaje(){
    conexion.value.send( JSON.stringify( mensaje.value ));
  }

  onMounted(async ()=>{

    conexion.value = new WebSocket( process.env.VUE_APP_API_URL )
    
    conexion.value.onmessage = function(event) {
      let msgRec = null
      try {
        msgRec = JSON.parse(event.data)
      } catch (error) {
        console.log(error)
      }
      if (msgRec !== null){
        mensajes.value.push( msgRec )
      }
    }

    conexion.value.onopen = function(event) {
      console.log(event)
      console.log("Successfully connected to the echo websocket server...")
    }
  })
</script>

<style scoped>
.mensaje-cont{
  height: 90vh;
  border: 1px solid #000;
  overflow-x: hidden;
  overflow-y: scroll;
}
</style>

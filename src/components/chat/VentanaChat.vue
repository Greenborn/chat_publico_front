<template>
  <div class="row">
    <div class="col p-3">

      <b-tabs content-class="mt-3">
        <b-tab 
          v-for="(chat) in chats_abiertos" :key="chat"
          :title="chat.titulo" :active="chat.activa">
          <div class="row">
          
            <div class="col col-sm-8 col-md-9 col-xxl-10 mensaje-cont">

              <div class="row" v-for="(msg) in chat.mensajes" :key="msg">
                <div class="col">
                  <span class="badge" :class="{ 'bg-secondary':msg.autor.id != datos_usuario.id, 'bg-primary': msg.autor.id == datos_usuario.id }">{{msg.autor.nombre}}:</span> {{msg.texto}}
                </div>
              </div>

            </div>

            <ListaContactos :online="chat.online" :datos_usuario="datos_usuario" @ir_sala_privada="ir_sala_privada"></ListaContactos>
          </div>

          <div class="row mt-3">
            <div class="col">
              <b-form-input
                    id="input-msg"
                    v-model="chat.mensaje.texto"
                    type="text" placeholder="Mensaje"
                    required></b-form-input>
            </div>

            <div class="col-auto">
              <b-button variant="success" @click="enviarMensaje( chat )">Enviar</b-button>
            </div>
          </div>

        </b-tab>
      </b-tabs>
     
    </div>
  </div>  

  <div class="app-modal h-100" v-if="modal.mostrar">
    <div class="row h-100">
      <div class="mt-auto mb-auto col-12 col-sm-8 offset-sm-2 col-md-6 offset-md-3 col-lg-4 offset-lg-4">

        <div class="card">
          <div class="card-header">
            <h5 class="card-title">Ingrese el nombre a mostrar</h5>
          </div>
          <div class="card-body">
            
            <div class="row mb-3">
              <div class="col">
                <b-form-input
                  id="input-nombre"
                  v-model="modelo_registro.nombre"
                  type="text" placeholder="Nombre"
                  required></b-form-input>
              </div>
            </div>

            <div class="row">
              <div class="col">
                <b-button variant="success" @click="registrarse( chats_abiertos[0] )">Registrarse</b-button>
              </div>
            </div>
  
          </div>
        </div>

      </div>
    </div>
  </div>
</template>

<script setup>
  import { ref, onMounted } from 'vue';
  import ListaContactos from './ListaContactos.vue'

  const chats_abiertos = ref([])

  const datos_usuario = ref({
    id: -1,
    nombre: 'Juan Perez'
  })

  const modelo_registro = ref({
    nombre: ''
  })

  const modal = ref({
    mostrar: false
  })

  function ir_sala_privada( usuario ){
    
  }

  function registrarse( chat ){
    let registro = {
      nombre: modelo_registro.value.nombre,
      accion: 'registro'
    }
    chat.conexion.send( JSON.stringify( registro ));
  }

  function enviarMensaje( chat ){
    if (chat.mensaje.texto !== ''){
      chat.mensaje.autor = datos_usuario.value
      chat.conexion.send( JSON.stringify( chat.mensaje ))
      chat.mensaje.texto = ''
    }
  }

  function inicializa_chat( chat ){
    chat.conexion = new WebSocket( import.meta.env.VITE_APP_API_URL )

    chat.conexion.onmessage = function(event) {
      let msgRec = null
      try {
        msgRec = JSON.parse(event.data)
      } catch (error) {
        console.log(error)
      }

      if (msgRec !== null){
        switch(msgRec.accion){
          case 'mensaje':
            chat.mensajes.push( msgRec )
          break;

          case 'mensaje_sys':
            chat.mensajes.push( {
              texto: msgRec.msg,
              accion: 'mensaje_sys',
              autor: {
                id: -1,
                nombre: 'Sistema'
              }
            } )
          break;

          case 'registro':
            datos_usuario.value = msgRec
            modal.value.mostrar = false
          break;

          case 'alerta':
            alert(msgRec.msg)
          break;

          case 'conectado':
            alert(msgRec.id)
          break;

          case 'reporte_online':
            chat.online = msgRec.reporte
          break;
        }
        
      }

    }

    chat.conexion.onopen = function(event) {
      console.log(event)
      console.log("Conectado a servidor de websocket...")
    }
  }

  onMounted(async ()=>{
    chats_abiertos.value.push({
      'id': '',
      'conexion': {},
      'activa': true,
      'online': [],
      'mensaje': {
        texto: '',
        accion: 'mensaje',
        autor: {}
      },
      'mensajes': [],
      'titulo': 'Chat general'
    })

    if (datos_usuario.value.id == -1){
      modal.value.mostrar = true
    }

    for (let c=0; c < chats_abiertos.value.length; c++){
      inicializa_chat( chats_abiertos.value[c] )
    }
    
  })
</script>

<style scoped>
.mensaje-cont{
  height: 70vh;
  border: 1px solid #000;
  overflow-x: hidden;
  overflow-y: scroll;
}

</style>

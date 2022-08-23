<template>
  <div class="row">
    <div class="col p-3">

      <div class="row">
        <div class="col mensaje-cont">

          <div class="row" v-for="(msg) in mensajes" :key="msg">
            <div class="col">
              <span class="badge" :class="{ 'bg-secondary':msg.autor.id != datos_usuario.id, 'bg-primary': msg.autor.id == datos_usuario.id }">{{msg.autor.nombre}}:</span> {{msg.texto}}
            </div>
          </div>

        </div>

        <div class="col-2">
          <div claSS="row conectados-tit">
            <div class="col">
              Conectados ({{online.length}})
            </div>
          </div>
          <div class="row" v-for="(user) in online" :key="user">
            <div class="col">
              {{user}}
            </div>
          </div>
        </div>
      </div>

      <div class="row mt-3">
        <div class="col">
          <b-form-input
                id="input-msg"
                v-model="mensaje.texto"
                type="text" placeholder="Mensaje"
                required></b-form-input>
        </div>

        <div class="col-auto">
          <b-button variant="success" @click="enviarMensaje()">Enviar</b-button>
        </div>
      </div>
     
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
                <b-button variant="success" @click="registrarse()">Registrarse</b-button>
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

  const mensajes  = ref([])
  const conexion  = ref({})
  const online    = ref([])

  const mensaje = ref({
    texto: '',
    accion: 'mensaje',
    autor: {}
  })

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

  function registrarse(){
    let registro = {
      nombre: modelo_registro.value.nombre,
      accion: 'registro'
    }
    conexion.value.send( JSON.stringify( registro ));
  }

  function enviarMensaje(){
    if (mensaje.value.texto !== ''){
      mensaje.value.autor = datos_usuario.value
      conexion.value.send( JSON.stringify( mensaje.value ))
      mensaje.value.texto = ''
    }
  }

  onMounted(async ()=>{

    if (datos_usuario.value.id == -1){
      modal.value.mostrar = true
    }

    conexion.value = new WebSocket( process.env.VUE_APP_API_URL )
    
    conexion.value.onmessage = function(event) {
      let msgRec = null
      try {
        msgRec = JSON.parse(event.data)
      } catch (error) {
        console.log(error)
      }

      if (msgRec !== null){
        switch(msgRec.accion){
          case 'mensaje':
            mensajes.value.push( msgRec )
          break;

          case 'mensaje_sys':
            mensajes.value.push( {
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
            online.value = msgRec.reporte
          break;
        }
        
      }

    }

    conexion.value.onopen = function(event) {
      console.log(event)
      console.log("Conectado a servidor de websocket...")
    }
  })
</script>

<style scoped>
.mensaje-cont{
  height: 75vh;
  border: 1px solid #000;
  overflow-x: hidden;
  overflow-y: scroll;
}

.conectados-tit{
  background: #198735;
  color: #FFF;
}
</style>

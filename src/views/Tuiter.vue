<template>
    <div class="mt-5">
        <h1>Tuiter</h1>
        <img id="img-tuiter" alt="Vue logo" src="@/assets/tuiterfake.png">
        <div class="container">
            <small>Bienvenido @{{userTuiter}}</small>
        </div>
        <b-alert
            :show="dismissCountDown"
            dismissible
            :variant="mensaje.color"
            @dismissed="dismissCountDown=0"
            @dismiss-count-down="countDownChanged"
            >
            {{mensaje.texto}}
        </b-alert>

        <div class="container">
            <form @submit.prevent="agregarTuiter()" v-if="!editar">
                <h3>Tuitea ahora</h3>
                <input type="text" class="form-control my-2" placeholder="Coloca el titulo de tu tuit." v-model="mensajes.descripcion">
                <input type="text" class="form-control my-2" placeholder="Escribe algo ..." v-model="mensajes.tuit">
                <b-button class="btn-block" variant="success" type="submit">Tuitear</b-button>

            </form>
        </div>
        <div class="container">
            <form @submit.prevent="editarTuit(mensajeEditable)" v-if="editar">
                <h3>Edita tu Tuit</h3>
                <input type="text" class="form-control my-2" placeholder="Coloca el titulo de tu tuit." v-model="mensajeEditable.descripcion">
                <input type="text" class="form-control my-2" placeholder="Escribe algo ..." v-model="mensajeEditable.tuit">
                <b-button class="btn-block" variant="warning" type="submit">Editar</b-button>
                <b-button class="btn-block" @click="editar = false" type="submit">Cancelar</b-button>
            </form>
        </div>

        <div class="container mt-5">
                    <div class="row">
                        <div class="col-6" v-for="(item, index) in almacen" :key="index">
                            <div class="card text-white bg-dark mb-3" style="max-width: 18rem;">
                                
                                <div class="card-header"><img id="img-principal" alt="Vue logo" src="../assets/userOfTuiter.jpeg">&nbsp;<img id="img-tuiter2" alt="Vue logo" src="@/assets/tuiterfake.png"> @{{userTuiter}}</div>
                                <div class="card-body">
                                    <h5 class="card-title">{{item.descripcion}}</h5>
                                    <p class="card-text">
                                        {{item.tuit}}
                                    </p>
                                    <b-button @click="eliminar(item._id)" 
                                    pill variant="danger">Borrar</b-button> &nbsp;
                                    <b-button @click="activarEditar(item._id)" 
                                    pill variant="warning">Editar</b-button>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
</template>

<script>
import { log } from 'util'
export default {
    name: 'Tuiter',
    data() {
        return {
            userTuiter:"Luis",
            almacen: [],
            dismissSecs: 5,
            dismissCountDown: 0,
            mensaje:{color:'', texto:''},
            mensajes:{tuit:'', descripcion:''},
            editar: false,
            mensajeEditable:{},
        }
    },
    created(){
        this.listarTuits();
    },
    methods: {
        alerta(){
            this.mensaje.color= 'danger';
            this.mensaje.texto= 'Probando Alerta';
            this.showAlert();
        },
        listarTuits(){
            this.axios.get('/tuit')
            .then(response => {
                console.log(response.data);
                this.almacen = response.data;
            })
            .catch(function (error) {
                console.log(error.response);
            })
        },
        agregarTuiter(){
            this.axios.post('/nuevo-tuit', this.mensajes)
            .then(response => {
                this.almacen.push(response.data)
                this.mensajes.tuit = '';
                this.mensajes.descripcion = '';
                this.mensaje.color= 'success';
                this.mensaje.texto= 'Has tuiteado!';
                this.showAlert();
            })
            .catch(error => {
                console.log(error.response);
                if (error.response.data.error.errors.tuit.message) {
                    this.mensaje.texto = error.response.data.error.errors.tuit.message;
                }else{
                    this.mensaje.texto = 'Error del aplicativo';
                }
                this.mensaje.color= 'danger';
                this.mensaje.texto= 'Vaya!, Parece que hubo un problema';
                this.showAlert();
            })
        },
        eliminar(id){
            console.log(id);
            this.axios.delete(`/tuit/${id}`)
            .then(response =>{
                const index = this.almacen.findIndex(item => item._id === response.data._id)
                this.almacen.splice(index, 1);
                this.mensaje.color= 'success';
                this.mensaje.texto= 'Eliminado satisfactoriamente!';
                this.showAlert();
            })
            .catch(e => {
                console.log(e.response);
                
            })
        },
        activarEditar(id){
            this.editar = true;
            console.log(id);
            this.axios.get(`/tuit/${id}`)
            .then(response =>{
              this.mensajeEditable = response.data
            })
            .catch(e => {
                console.log(e.response);
                
            })
            
        },
        editarTuit(item){
            this.axios.put(`/tuit/${item._id}`, item)
            .then(response =>{
                const index = this.mensajes.findIndex(n => n._id === response.data._id);
                this.mensajes[index].tuit = response.data.tuit;
                this.mensajes[index].descripcion = response.data.descripcion;
                this.mensajeEditable = {}
                this.mensaje.color= 'success';
                this.mensaje.texto= 'Editado satisfactoriamente!';
                this.showAlert();
                this.editar = false;
            })
            .catch(e => {
                console.log(e.response);
                
            })
        },
        countDownChanged(dismissCountDown) {
        this.dismissCountDown = dismissCountDown
        },
        showAlert() {
        this.dismissCountDown = this.dismissSecs
        }

    }
}
</script>

<style>
    #img-tuiter{
        width: 100px;
    }
    #img-tuiter2{
        width: 20px;
    }
    #img-principal{
        width: 50px;
        border-radius: 30px
    }
</style>
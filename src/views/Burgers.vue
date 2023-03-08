<template>
  <div>
    <v-container>
      <v-card>
        <v-card-title>Mantenedor de Hamburguesas</v-card-title>
        <v-form class="mt-5 ml-5 mr-5">
          <v-row>
            <v-col class="mt-n5" cols="12" md="4">
              <v-alert class='text-caption' dense text color="success">Nombre Hamburguesa</v-alert>
            </v-col>
            <v-col class="mt-n5" cols="12" md="4">
              <v-text-field class='text-caption' v-model="nombre" placeholder="Ejemplo: Barros Luco" outlined dense  clearable></v-text-field>
            </v-col>
          </v-row>
          <v-row>
            <v-col class="mt-n5" cols="12" md="4">
              <v-alert class='text-caption' dense text color="success">Ingredientes</v-alert>
            </v-col>
            <v-col class="mt-n5" cols="12" md="4">
              <v-text-field class='text-caption' v-model="ing_01" placeholder="Ingrediente 01" outlined dense  clearable></v-text-field>
            </v-col>
            <v-col class="mt-n5" cols="12" md="4">
              <v-text-field class='text-caption' v-model="ing_02" placeholder="Ingrediente 02" outlined dense  clearable></v-text-field>
            </v-col>
          </v-row>
          <v-row>
            <v-col class="mt-n5" cols="12" md="4">
              <v-alert class='text-caption' dense text color="success">Calorias</v-alert>
            </v-col>
            <v-col class="mt-n5" cols="12" md="4">
              <v-text-field class='text-caption' v-model="calorias" type="number" placeholder="Solo Numeros" outlined dense  clearable></v-text-field>
            </v-col>
            <v-col class="mt-n5" cols="12" md="4">
              <v-btn v-if="existe!='si'" @click="agregar()" class="text-caption" color="success" dark block>Agregar Hamburguesa</v-btn>
              <v-btn v-else @click="modificar()" class="text-caption" color="success" dark block>Modificar Hamburguesa</v-btn>
            </v-col>
          </v-row>
        </v-form>
        <v-divider class="mt-5"></v-divider>
        <v-card-actions>
          <v-btn color="success" fab dark small>{{total}}</v-btn>
          <h3 class="ml-2 text-caption">Total</h3>
          <v-spacer></v-spacer>
            <v-btn @click="ver_lista()" class="text-caption" color="success" text>ver lista</v-btn>
        </v-card-actions>
      </v-card>
    </v-container>
    <v-dialog v-model="dialogo" transition="dialog-bottom-transition" max-width="80%" persistent>
      <v-card>
          <v-card-title>
              <v-btn @click="dialogo=false" color="red" small dark><v-icon small>close</v-icon>Cerrar</v-btn>
          </v-card-title>
          <v-card-text>
              <v-data-table dark dense :headers="titulos" :items="lista" :page.sync="page" :items-per-page="itemsPerPage" hide-default-footer class="elevation-1" @page-count="pageCount = $event">
                  <template v-slot:item.id="{ item }">
                    <v-chip small color="success" dark>
                        {{ item.id }}
                    </v-chip> 
                  </template>
                  <template v-slot:item.edita="{ item }">
                      <v-icon small @click="editar(item)">edit</v-icon>
                  </template>
                  <template v-slot:item.elimina="{ item }">
                      <v-icon small @click="confirmar(item.id)">delete</v-icon>
                  </template>
              </v-data-table>
              <v-pagination v-model="page" :length="pageCount"></v-pagination>
          </v-card-text>
      </v-card>
    </v-dialog>
    <v-dialog v-model="confirma" transition="dialog-bottom-transition" max-width="50%" persistent>
      <v-container>
      <v-card>
        <v-card-text>
          <strong>Esta Seguro que desea eliminar la hamburguesa seleccionada?</strong>
        </v-card-text>
        <v-card-actions>
          <v-btn @click="eliminar(registro)" class="text-caption" color="success">si</v-btn>
          <v-spacer></v-spacer>
          <v-btn @click="confirma=false" class="text-caption" color="error">no</v-btn>
        </v-card-actions>
      </v-card>
      </v-container>
    </v-dialog>
    <v-snackbar v-model="snack" :timeout="2000" color="blue-grey" rounded="pill">{{mensaje}}</v-snackbar>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  name: 'Burgers',
  data(){
    return{
      snack:false,
      mensaje:'',
      ids:0,
      nombre:'',
      ing_01:'',
      ing_02:'',
      calorias:'',
      total:0,
      dialogo:false,
      titulos: [
        { text: 'id', align: 'center', sortable: false, value: 'id' },
        { text: 'Nombre', align: 'start', sortable: true, value: 'hamburguesa' },
        { text: 'Ingrediente 1', align: 'start', sortable: false, value: 'ingrediente_01' },
        { text: 'Ingrediente 2', align: 'start', sortable: false, value: 'ingrediente_02' },
        { text: 'Calorias', align: 'center', sortable: true, value: 'caloria' },
        { text: 'Editar', align: 'center', sortable: false, value: 'edita' },
        { text: 'Eliminar', align: 'center', sortable: false, value: 'elimina' },
      ],
      lista:[],
      page: 1,
      pageCount: 0,
      itemsPerPage: 8,
      existe:'no',
      confirma:false,
      registro:null
    }
  },
  mounted(){
    this.leer_lista()
  },
  methods:{
    limpia(){
      this.ids=0;
      this.nombre='';
      this.ing_01='';
      this.ing_02='';
      this.calorias='';
      this.existe='no';
      this.mensaje='';
      this.snack=false;
      this.registro=null
    },
    async ver_lista(){
      await this.leer_lista()
      this.dialogo=true;
    },
    async leer_lista(){
      try {
        const headers = {
          'Content-Type': 'application/json',
        };
        const response = await axios.get('https://hamburguesas-back.elevadev.cl/burger/',{headers});
        const datos=response.data;
        //console.log(datos);
        if(datos.length!=0){
          this.lista=[];
          let reg=0;
          datos.forEach(element => {
            this.lista.push({
              id:element.id,
              hamburguesa:element.nombre,
              ingrediente_01:element.ingredientes[0],
              ingrediente_02:element.ingredientes[1],
              caloria:element.calorias
            });
            reg++            
          });
          this.total=reg;
        }
      } catch (error) {
        console.log(error)        
      }
    },
    async agregar(){
      try {
        const headers = {
          'Content-Type': 'application/json',
        };
        const params= {
          'nombre':this.nombre,
          'ingredientes':[this.ing_01,this.ing_02],
          'calorias':this.calorias
        };
        const response = await axios.post('https://hamburguesas-back.elevadev.cl/burger/',params,{headers});
        const datos=response.data;
        if(datos.length!=0){ 
          this.total++;
          this.limpia();
          this.mensaje="Hamburguesa Agregada";
          this.snack=true
         }
      } catch (error) {
        console.log(error)
      }
    },
    async editar(e){
      this.ids=e.id;
      this.nombre=e.hamburguesa;
      this.ing_01=e.ingrediente_01;
      this.ing_02=e.ingrediente_02;
      this.calorias=e.caloria;
      this.existe='si';
      this.dialogo=false;
    },
    confirmar(e){
      this.confirma=true;
      this.registro=e;
    },
    async modificar(){
      try {

        // se intento de hacer un PUT y un PATCH no dando resultados positivos:: error 500

        this.existe='no';
        this.limpia();
        this.mensaje="Hamburguesa Modificada";
        this.snack=true

        // const headers = {
        //   'Content-Type': 'application/json',
        // };
        // const params= {
        //   'id':this.ids,
        //   'nombre':this.nombre,
        //   'ingredientes':[this.ing_01,this.ing_02],
        //   'calorias':this.calorias
        // };
        // const response = await axios.patch('https://hamburguesas-back.elevadev.cl/burger',params,{headers});
        // const datos=response.data;
        // console.log(datos);        
      } catch (error) {
        console.log(error)
      }
    },
    async eliminar(e){
      try {
        const headers = {
          'Content-Type': 'application/json',
        };
        const response = await axios.delete('https://hamburguesas-back.elevadev.cl/burger/'+e,{headers});
        const datos=response.data;
        this.mensaje="Hamburguesa Eliminada";
        this.confirma=false;
        this.snack=true
        this.leer_lista();
      } catch (error) {
        console.log(error)
      }
    }
  }
}
</script>

<style>
</style>

<template>
    <Navbar/>
      <!-- ////////// formulario Añadir ////////// -->
    <!-- Nombre -->
  <div class="container my-4">
  <form>  
    <div class="input-group mb-3">
    <span class="input-group-text">Nombre</span>
    <input v-model="producto.nombre" type="text" class="form-control">
    </div>
    <!-- Correo -->
    <div class="input-group mb-3">
    <span class="input-group-text">Correo</span>
    <input v-model="producto.precio" type="text" class="form-control">
    </div>
    <div class="input-group my-3">
      <input type="file" @change="buscarImagen($event)">
    </div>

      <div class="mt-3">  
    <button v-show="this.editar === true" 
      @click.prevent="actualizarDato(id)" 
      class="btn btn-primary">
      Actualizar
    </button>
    <button v-show="this.editar === false" 
      @click.prevent="agregarDato()" 
      class="btn btn-primary">
      Guardar
    </button>
    <div class="mt-4">
      <img :src="datoImagen">
    </div>

    </div>
  </form>

<!-- ////////// tabla ////////// -->
  <table class="table">
    <thead>
      <tr>
        <th scope="col">id</th>
        <th scope="col">Producto</th>
        <th scope="col">Familia</th>
        <th scope="col">Editar</th>
        <th scope="col">Eliminar</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="(item, index) in productos" :key="index">
        <th scope="row">{{index}}</th>
        <td>{{item.nombre}}</td>
        <td>{{item.correo}}</td> 
        <td>
          <button @click.prevent="obtenerDatoID( item.id );this.editar = !this.editar;" 
            class="btn btn-primary">Editar
          </button>
        </td>

        <td>
          <button @click.prevent="eliminarDato(item.id)" 
            class="btn btn-danger">Eliminar
          </button>
      </td>
      </tr>
    </tbody>
  </table>
  </div>
</template>

<script>
import Navbar from '@/components/Navbar.vue'
import { collection, getDocs, addDoc, deleteDoc, doc, getDoc, updateDoc  } from 'firebase/firestore/lite';
import { db, storage } from "../main";
import firebase from 'firebase/compat/app';
import router from '../router/index'

export default {
  name: 'About',
  components: {
    Navbar
  },
  data() {
    return {
      file: null,
      datoImagen: null,
      error: null,
      editar: false,
      loading: false,

      productos: [],
      producto: {
          id: '',
          nombre: '',
          familia: '',
          descripcion: '',
          precio: '',
          valoracion: '',
          unidades: '',
          enlace: '',
          foto: ''}
  }
  },
  methods: {
    async obtenerDatos () { 
      const querySnapshot = await getDocs(collection(db, "productos"));
        querySnapshot.forEach((doc) => {
        let producto = doc.data()
        producto.id = doc.id
        this.productos.push(producto)
        console.log(producto)
      });
    },
    // DELETE / ELIMINAR / BORRAR
    async eliminarDato (id){
      await deleteDoc(doc(db, "productos", id ));
      router.go('/')
    },
    // GET BY ID / OBTENER POR ID
    async obtenerDatoID (id){
      const docRef = doc(db, "productos", id);
      const docSnap = await getDoc(docRef);
          if (docSnap.exists()) {
            this.producto = docSnap.data()
            this.producto.id = docSnap.id
            } 
            else {
            console.log("¡No existe el documento!");
            }
    },

     // BUSCAR IMAGEN
    buscarImagen(event){
      const tipoArchivo = event.target.files[0].type;
      if(tipoArchivo === 'image/jpeg' || tipoArchivo === 'image/png'){
          this.file = event.target.files[0]
          this.error = null
      }
          else{
          this.error = 'Archivo no válido'
          this.file = null
          return 
          }
          const reader = new FileReader();
          reader.readAsDataURL(this.file);
          reader.onload = (e) => {
          this.datoImagen = e.target.result
          }
    },
    // SUBIR IMAGEN STORAGE
  async agregarDato(){
    try {
     this.loading = true
      var storageRef = firebase.storage().ref();
      await storageRef.child('imagenes').child(this.file.name).put(this.file)
      const urlDescarga = await storageRef.child('imagenes').child(this.file.name).getDownloadURL()
      await addDoc(collection(db, "productos"), {
          nombre: this.producto.nombre,
          descripcion: this.producto.descripcion,
          precio: this.producto.precio,
          valoracion: this.producto.valoracion,
          familia: this.producto.familia,
          unidades: this.producto.unidades,
          enlace: this.producto.enlace,
          foto: urlDescarga
        })
        this.error = 'Imagen subida con éxito'
        this.file = null
    } 
      catch (error) {
        console.log(error);
      } 
      finally {
        router.push('/')
        this.loading = false
      }
    },

// MÉTODO actualizarDato
async actualizarDato(){
    try {
      this.loading = true
      var storageRef = firebase.storage().ref();
      await storageRef.child('imagenes').child(this.file.name).put(this.file)
      const urlDescarga = await storageRef.child('imagenes').child(this.file.name).getDownloadURL()
      const elemento = doc(db, "productos", this.producto.id )
      await updateDoc(elemento, {
          nombre: this.producto.nombre,
          descripcion: this.producto.descripcion,
          precio: this.producto.precio,
          valoracion: this.producto.valoracion,
          familia: this.producto.familia,
          unidades: this.producto.unidades,
          enlace: this.producto.enlace,
          foto: urlDescarga
        })
        this.error = 'Imagen subida con éxito'
        this.file = null
    } 
      catch (error) {
        console.log(error);
      } 
      finally {
        router.push('/')
        this.loading = false
      }}
  },
  mounted() {
    this.obtenerDatos();
  },
}

</script>
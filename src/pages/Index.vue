<template>
  <q-page class="flex flex-center">
      <div class="q-pa-md">
        <q-table
          title="Productos"
          :rows="serverData"
          :columns="columns"
          row-key="name"
          @request="getProductos"
        >
          <template v-slot:body="props">
            <q-tr :props="props">
              <q-td>
                {{ props.row.id }}
              </q-td>
              <q-td>
                {{ props.row.nombre }}
              </q-td>
              <q-td>
                {{ props.row.precio.toFixed(2) }}
              </q-td>
              <q-td>
                {{ props.row.categoria }}
              </q-td>
              <q-td>
                {{ props.row.disponible }}
              </q-td>
              <q-td @click="openEdit(props.row.id)">
                <img v-if="props.row.imagen" v-bind:src="props.row.imagen" width="100"/>
                <img v-else src="../assets/default.svg" width="100">
              </q-td>
            </q-tr>
          </template>
        </q-table>
      </div>
      <q-dialog v-model="layoutModalimg" persistent>
      <q-card class="my-card" flat bordered style="width: 100%;">
        <q-card-section style="display: flex;height: 350px; justify-content: center; align-items: center;height: 350px;text-align: center;">
          <q-spinner
            color="primary"
            size="3em"
            v-if="loaderimg"
          />
          <img v-if="noimg" src="../assets/default.svg" style="height: 50%;">
        </q-card-section>
        <q-card-section>
          <q-form @submit.prevent="onSubmit" class="q-gutter-md">
            <q-input
                name="inputId"
                id="inputId"
                type="hidden"
                v-model="idproductoUpd"
            />
            <q-input
                @update:model-value="val => { file = val[0] }"
                filled
                name="inputImg"
                id="inputImg"
                type="file"
            />
            <div>
                <q-btn label="Enviar" type="submit" color="primary" style="margin: 20px;"/>
                <q-btn label="Cerrar" type="button" color="secondary" v-close-popup style="margin: 20px;"/>
            </div>
          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script>

import { defineComponent, ref } from 'vue'
import axios from 'axios'
// const ENDPOINT_PATH = process.env.BACKEND_V2_URL
// const ENDPOINT_PATH = 'https://ejdevelop.com/bloque7_backend/'
const ENDPOINT_PATH = 'http://localhost:4001/'

const columns = [
  { name: 'id', align: 'center', label: 'ID', field: 'id', sortable: true },
  {
    name: 'nombre',
    required: true,
    label: 'Nombre',
    align: 'left',
    field: row => row.nombre,
    format: val => `${val}`,
    sortable: true
  },
  { name: 'precio', align: 'center', label: 'Precio', field: 'precio', sortable: true },
  { name: 'categoria', label: 'Categoria', field: 'categoria', sortable: true },
  { name: 'disponible', label: 'Disponible', field: 'disponible' },
  { name: 'imagen', label: 'Imagen', field: 'imagen', sortable: true }
]

export default defineComponent({
  name: 'PageIndex',
  data () {
    return {
      serverData: []
    }
  },
  setup () {
    const urlupload = ref(ENDPOINT_PATH + 'upload')
    const idproductoUpd = ref(false)
    const layoutModalimg = ref(false)
    const loaderimg = ref(false)
    const noimg = ref(true)
    return {
      columns,
      loaderimg,
      noimg,
      layoutModalimg,
      urlupload,
      idproductoUpd
    }
  },
  methods: {
    openEdit (id) {
      this.layoutModalimg = true
      this.idproductoUpd = id
      console.log(this.idproductoUpd)
    },
    async onSubmit (evt) {
      const formData = new FormData(evt.target)
      console.log(this.idproductoUpd)
      formData.append('nombreimagen', this.idproductoUpd)
      await axios.post(this.urlupload, formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      }).then(function (response) {
        console.log(response.data)
        // this.getProductos()
      }).catch(function (error) {
        console.log('error:> ', error)
      })
      // await this.getProductos()
      this.$router.go(0)
      this.layoutModalimg = false
    },
    async getProductos () {
      const data = { categoria: null }
      const resp = await axios.post(ENDPOINT_PATH + 'listarproductos', data)
      this.serverData = []
      const datos = resp.data
      console.log(datos)
      for (const i in datos) {
        const item = datos[i]
        const obj = {}
        obj.id = item.id
        obj.nombre = item.nombre
        obj.precio =
          item.porkilos === 1
            ? item.precio
            : parseFloat(item.precio / item.unixcaja)
        obj.disponible = item.disponible
        obj.categoria = item.idcategoria
        let img = null
        // console.log(ENDPOINT_PATH + 'files/' + item.id + '.png')
        if (this.fileExists(ENDPOINT_PATH + 'files/' + item.id + '.png')) {
          img = ENDPOINT_PATH + 'files/' + item.id + '.png'
        }
        console.log(img)
        obj.imagen = img
        this.serverData.push(obj)
      }
    },
    fileExists (path) {
      const http = new XMLHttpRequest()
      http.open('HEAD', path, false)
      http.send()
      return http.status !== 404
    }
  },
  mounted () {
    this.getProductos()
  }
})
</script>

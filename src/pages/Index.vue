<template>
  <q-page class="flex flex-center">
      <div class="q-pa-md">
        <q-table
          title="Productos"
          :rows="serverData"
          :columns="columns"
          row-key="id"
          :key="tableKey"
          :pagination="initialPagination"
          :filter="filter"
        >
          <template v-slot:top-right>
            <q-input borderless dense debounce="300" v-model="filter" placeholder="Search">
              <template v-slot:append>
                <q-icon name="search" />
              </template>
            </q-input>
          </template>
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
              <q-td>
                <img v-if="props.row.imagen" v-bind:src="props.row.imagen" width="100"/>
                <img v-else src="../assets/default.svg" width="100">
              </q-td>
              <q-td>
                <q-btn v-if="props.row.imagen" @click="onDelete(props.row.id)" label="Eliminar imagen" v-close-popup type="buttom" color="negative" style="margin: 20px;"/>
                <q-btn v-else @click="openEdit(props.row.id)" label="Agregar imagen" v-close-popup type="buttom" color="primary" style="margin: 20px;"/>
              </q-td>
            </q-tr>
          </template>
        </q-table>
      </div>
      <!-- <q-dialog v-model="layoutModalimg" persistent>
        <q-uploader
        :url="urlupload"
        style="max-width: 300px"
      />
      </q-dialog> -->
      <q-dialog v-model="layoutModalimg" persistent>
      <q-card class="my-card" flat bordered style="width: 100%;">
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
                <q-btn label="Guardar imagen" type="submit" color="primary" style="margin: 20px;"/>
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
const config = require('../config/endpoints.js')
const ENDPOINT_PATH = config.endpoint_path

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
  { name: 'imagen', label: 'Imagen', field: 'imagen', align: 'center', sortable: true },
  { name: 'editar', label: 'Editar', align: 'center' }
]

export default defineComponent({
  name: 'PageIndex',
  data () {
    return {
      serverData: [],
      datos: [],
      tableKey: 0
    }
  },
  setup () {
    const urlupload = ref(ENDPOINT_PATH + 'upload')
    const idproductoUpd = ref(false)
    const layoutModalimg = ref(false)
    const loaderimg = ref(false)
    const noimg = ref(true)
    const filter = ref('')
    return {
      filter,
      initialPagination: {
        sortBy: 'desc',
        descending: false,
        page: 1,
        rowsPerPage: 50
        // rowsNumber: xx if getting data from a server
      },
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
      // this.$router.go(0)
      this.layoutModalimg = false
      this.reloadProducto(this.datos)
      // await this.getProductos()
    },
    async onDelete (id) {
      console.log(id)
      this.$q.dialog({
        title: 'CONFIRMACIÓN!',
        message: 'Seguro que desea ELIMINAR esta IMAGEN?',
        ok: {
          color: 'primary',
          label: 'Sí'
        },
        cancel: {
          color: 'secondary',
          label: 'No'
        },
        persistent: true
      }).onOk(async () => {
        const data = {
          img: id + '.png'
        }
        await axios.post(ENDPOINT_PATH + 'deletefiles', data).then(function (response) {
          console.log(response.data)
          // this.getProductos()
        }).catch(function (error) {
          console.log('error:> ', error)
        })
        // this.$router.go(0)
        this.reloadProducto(this.datos)
      })
      this.layoutModalimg = false
      // await this.getProductos()
    },
    async getProductos () {
      const data = { categoria: null }
      const resp = await axios.post(ENDPOINT_PATH + 'listarproductos', data)
      this.datos = resp.data
      console.log('getProductos')
      this.reloadProducto(this.datos)
    },
    async reloadProducto (datos) {
      this.serverData = []
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
        const resp = await axios.get(ENDPOINT_PATH + 'files/' + item.id + '.png')
        if (resp.status === 200) {
          img = ENDPOINT_PATH + 'files/' + item.id + '.png'
        }
        obj.imagen = img
        this.serverData.push(obj)
      }
    }
  },
  mounted () {
    // this.$emit('edit', this.serverData)
    this.getProductos()
  }
})
</script>

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
          <template v-slot:top-left>
            <q-btn
              dense label ="migrar todos"
              color="primary"
              icon="update"
              class="q-ml-md"
              @click="migrarAll"
            />
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
                <img v-if="!props.row.imagen" src="../assets/default.svg" width="100">
                <img v-else-if="props.row.imagen.length > 10" v-bind:src="props.row.imagen" width="100"/>
                <img v-else src="../assets/default.svg" width="100">

              </q-td>
              <q-td>
                <q-btn v-if="!props.row.imagen" @click="openEdit(props.row.id)" label="Agregar imagen" v-close-popup type="buttom" color="primary" style="margin: 20px;"/>
                <q-btn v-else-if="props.row.imagen.length > 10" @click="onDelete(props.row.id)" label="Eliminar imagen" v-close-popup type="buttom" color="negative" style="margin: 20px;"/>
                <q-btn v-else @click="openMigrar(props.row.id, props.row.nombre)" label="Migrar imagen" v-close-popup type="buttom" color="info" style="margin: 20px;"/>
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
    <!-- VER IMAGEN A MIGRAR-->
    <q-dialog v-model="layoutModalimgupload" persistent>
      <q-card class="my-card" flat bordered style="width: 100%;">
        <q-card-section style="display: flex;height: 350px; justify-content: center; align-items: center;height: 350px;text-align: center;">
          <q-spinner
            color="primary"
            size="3em"
            v-if="loaderimg"
          />
          <img v-if="imagenproductoimg" :src="imagenproductoimg" style="height: 90%;">
          <div v-if="subtituloimg" class="subtitleimagenPopup">
            {{ nombreproductoimg }}
          </div>
        </q-card-section>
        <q-card-actions align="center" class="text-primary">
          <q-btn
            style=""
            label="Cerrar"
            type="buttom"
            color="primary"
            icon="close"
            v-close-popup
          />
        </q-card-actions>
      </q-card>
    </q-dialog>
    <div v-if="loader" class="procesando">
      <span style="display: grid"
        >Migrando, espere...
        <q-linear-progress indeterminate />
      </span>
    </div>
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
    const layoutModalimgupload = ref(false)
    const loaderimg = ref(false)
    const loader = ref(false)
    const noimg = ref(true)
    const filter = ref('')
    const subtituloimg = ref(false)
    return {
      filter,
      subtituloimg,
      initialPagination: {
        sortBy: 'desc',
        descending: false,
        page: 1,
        rowsPerPage: 50
        // rowsNumber: xx if getting data from a server
      },
      columns,
      loader,
      loaderimg,
      noimg,
      layoutModalimg,
      layoutModalimgupload,
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
    async openMigrar (id, nombre) {
      this.layoutModalimgupload = true
      this.idproductoUpd = id
      console.log(this.idproductoUpd)
      this.loaderimg = true
      this.imagenproductoimg = false
      this.nombreproductoimg = false
      const resp = await this.getimagenproducto(id)
      this.imagenproductoimg = resp.data.length > 0 ? this.dataUrl(resp.data[0].imagen) : null
      this.loaderimg = false
      this.nombreproductoimg = nombre
      this.reloadProducto(this.datos)
    },
    async migrarAll () {
      console.log('Inicio migrar')
      this.loader = true
      for (const i in this.datos) {
        const item = this.datos[i]
        if (item.imagen) {
          const resp = await axios.get(ENDPOINT_PATH + 'files/' + item.id + '.png')
          if (resp.status !== 200) {
            await this.getimagenproducto(item.id)
            console.log(item.id, resp.status)
          }
        }
      }
      console.log('Fin migrar')
      this.loader = false
      this.reloadProducto(this.datos)
    },
    getimagenproducto (idproducto) {
      const data = { idproducto }
      return axios.post(ENDPOINT_PATH + 'getimagenproducto', data)
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
      const resp = await axios.post(ENDPOINT_PATH + 'listarproductosimg', data)
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
        // let img = null
        obj.imagen = item.imagen
        const resp = await axios.get(ENDPOINT_PATH + 'files/' + item.id + '.png')
        if (resp.status === 200) {
          obj.imagen = ENDPOINT_PATH + 'files/' + item.id + '.png'
        }
        this.serverData.push(obj)
      }
    },
    dataUrl (img) {
      // console.log(img.data)
      return 'data:image/png;base64,' + btoa(
        new Uint8Array(img.data).reduce((data, byte) => data + String.fromCharCode(byte), '')
      )
    }
  },
  mounted () {
    // this.$emit('edit', this.serverData)
    this.getProductos()
  }
})
</script>
<style>
  .subtitleimagenPopup {
    text-align: center;
    pointer-events: all;
    padding: 10px;
    color: #fff;
    background: rgba(0, 0, 0, 0.47);
    font-size: 12px;
    position: absolute;
    top: 10px;
    right: 10px;
    left: 10px;
  }
  .procesando {
    background: #3a4142cc;
    width: 100%;
    height: 100%;
    position: fixed;
    z-index: 99999;
    top: 0px;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 25px;
    color: white;
  }
</style>

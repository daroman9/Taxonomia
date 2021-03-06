<template>
  <v-layout align-start>
    <v-flex>
      <v-toolbar flat color="white">
        <v-toolbar-title>Maestros</v-toolbar-title>
        <v-divider class="mx-2" inset vertical></v-divider>
        <v-spacer></v-spacer>
        <v-text-field
          class="text-xs-center"
          v-model="search"
          append-icon="search"
          label="Búsqueda"
          single-line
          hide-details
        ></v-text-field>
        <v-spacer></v-spacer>
        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on }">
            <v-btn color="primary" dark class="mb-2" v-on="on">Nuevo</v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="headline">{{ formTitle }}</span>
            </v-card-title>
            <v-card-text>
              <v-container grid-list-md>
                <v-layout wrap>
                  <v-flex xs12 sm6 md12>
                    <v-text-field v-model="nombre" label="Nombre"></v-text-field>
                  </v-flex>
                  <v-spacer></v-spacer>
                  <v-flex xs12 sm6 md12 v-show="valida">
                    <div class="red--text" v-for="v in validaMensaje" :key="v" v-text="v"></div>
                  </v-flex>
                </v-layout>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" flat @click="close">Cancelar</v-btn>
              <v-btn color="blue darken-1" flat @click="guardar">Guardar</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>

        <v-dialog v-model="dialogEliminar" max-width="500px">
          <v-card>
            <v-card-title>
              <span class="headline">Eliminar Maestro</span>
            </v-card-title>
            <v-card-text>
              <v-container grid-list-md>
                <v-layout wrap>
                  <v-flex xs12 sm6 md12>
                    <v-card-text>
                      ¿Estas seguro que deseas eliminar el
                      Maestro?
                    </v-card-text>
                  </v-flex>
                  <v-spacer></v-spacer>
                  <v-flex xs12 sm6 md12 class="red--text" v-if="errorM">{{ errorM }}</v-flex>
                </v-layout>
              </v-container>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" flat @click="closeEliminar">Cancelar</v-btn>
              <v-btn color="blue darken-1" flat @click="eliminar">Guardar</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
      <v-data-table :headers="headers" :items="plantillas" :search="search" class="elevation-1">
        <template v-slot:items="props">
          <td>{{ props.item.id }}</td>
          <td>{{ props.item.nombre }}</td>
          <td class="justify-left layout px-10">
            <v-icon small class="mr-2" @click="editItem(props.item)">edit</v-icon>
            <v-icon small class="mr-2" @click="eliminarItem(props.item)">delete</v-icon>
          </td>
        </template>
        <template v-slot:no-data>
          <v-btn color="primary" @click="listar()">Resetear</v-btn>
        </template>
      </v-data-table>
    </v-flex>
  </v-layout>
</template>
<script>
import axios from "axios";
export default {
  data() {
    return {
      dialog: false,
      dialogEliminar: false,
      search: "",
      _id: "",
      nombre: "",
      valida: 0,
      validaMensaje: [],
      plantillas: [],
      errorM: null,
      headers: [
        { text: "Id", value: "id", sortable: true },
        { text: "Nombre", value: "nombre" },
        { text: "Opciones", value: "Opciones", sortable: false }
      ],
      deleteItem: -1,
      editedIndex: -1,
      editedItem: {
        name: ""
      },
      defaultItem: {
        name: ""
      }
    };
  },
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "Nuevo Maestro" : "Editar Maestro";
    }
  },
  watch: {
    dialog(val) {
      val || this.close();
    }
  },
  created() {
    this.listar();
  },
  methods: {
    listar() {
      let me = this;
      axios
        .get("selectors")
        .then(function(response) {
          me.plantillas = response.data;
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    limpiar() {
      (this._id = ""),
        (this.nombre = ""),
        (this.imagenUrl = ""),
        (this.validaMensaje = []),
        (this.editedIndex = -1);
    },
    validar() {
      this.valida = 0;
      this.validaMensaje = [];
      if (this.nombre.length < 1 || this.nombre.length > 50) {
        this.validaMensaje.push(
          "El nombre del Maestro debe tener entre 1 y 50 caracteres"
        );
      }

      if (this.validaMensaje.length) {
        this.valida = 1;
      }
      return this.valida;
    },
    guardar() {
      let me = this;
      if (this.validar()) {
        return;
      }
      if (this.editedIndex > -1) {
        //Codigo para editar una plantilla
        axios
          .put("selectors/" + this._id, {
            id: this._id,
            nombre: this.nombre,
            imageUrl: this.imageUrl
          })
          .then(function(response) {
            me.limpiar();
            me.close();
            me.listar();
          })
          .catch(function(error) {
            console.log(error);
          });
      } else {
        //Codigo para guardar una nueva plantilla
        axios
          .post("selectors", {
            nombre: this.nombre,
            imageUrl: this.imageUrl
          })
          .then(function(response) {
            me.limpiar();
            me.close();
            me.listar();
          })
          .catch(function(error) {
            console.log(error);
          });
      }
    },
    eliminar() {
      let me = this;
      axios
        .delete("selectors/" + this._id, {})
        .then(function(response) {
          me.limpiar();
          me.closeEliminar();
          me.listar();
        })
        .catch(error => {
          this.errorM = "No se puede eliminar este registro";
        });
    },
    mostrarmensaje() {
      this.dialogEliminar = false;
    },

    editItem(item) {
      this._id = item.id;
      this.nombre = item.nombre;
      this.imageUrl = item.imageUrl;
      this.dialog = true;
      this.editedIndex = 1;
    },
    eliminarItem(item) {
      this._id = item.id;
      this.dialogEliminar = true;
      this.deleteIndex = 1;
    },
    close() {
      this.dialog = false;
    },
    closeEliminar() {
      this.dialogEliminar = false;
    }
  }
};
</script>

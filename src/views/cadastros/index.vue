<template>
  <div>
    <el-menu :default-active="$route.path" :router="true" class="el-menu-demo" mode="horizontal" background-color="#545c64" text-color="#fff" active-text-color="#ffd04b">
      <el-menu-item :index="'/cadastros/index/'+tipoCadastro" :route="'/cadastros/index/'+tipoCadastro">Ver Todos</el-menu-item>
      <el-menu-item :index="'/cadastros/add/'+tipoCadastro" :route="'/cadastros/add/'+tipoCadastro">Novo Cadastro</el-menu-item>
    </el-menu>

    <div class="line"/>
    <tabela-cliente v-if="$route.params.tipo == 'clientes'"/>
    <tabela-especie v-if="$route.params.tipo == 'especies'"/>
    <tabela-raca v-if="$route.params.tipo == 'racas'"/>
    <tabela-exame v-if="$route.params.tipo == 'exames'"/>

    <el-dialog :visible.sync="formVisible" title="Editar">
      <form-especies v-if="$route.params.tipo == 'especies'" :form="form"/>
      <form-raca v-if="$route.params.tipo == 'racas'" :form="form"/>
      <form-exame v-if="$route.params.tipo == 'exames'" :form="form"/>
      <span slot="footer" class="dialog-footer">
        <el-button @click="formVisible = false">Cancelar</el-button>
        <el-button type="primary" @click="saveDados">Salvar</el-button>
      </span>
    </el-dialog>

  </div>

</template>

<script>

import axios from 'axios'

import tabelaCliente from './tabelas/cliente'
// import tabelaPadrao from './tabelas/padrao'
import tabelaEspecie from './tabelas/especie.vue'
import tabelaRaca from './tabelas/raca.vue'
import tabelaExame from './tabelas/exame.vue'

/** * Formularios ***/
import formEspecie from './forms/especies.vue'
import formCliente from './forms/clientes.vue'
import formRaca from './forms/raca.vue'
import formExame from './forms/exames.vue'

import AwesomeMask from 'awesome-mask'
import localForage from 'localforage'
import moment from 'moment'

export default {
  name: 'CadastrosIndex',
  directives: {
    'mask': AwesomeMask
  },
  filters: {
    moment: function(date) {
      return moment(date).format('MMMM Do YYYY, h:mm:ss a')
    }
  },
  components: {
    'tabela-cliente': tabelaCliente,
    'tabela-especie': tabelaEspecie,
    'tabela-raca': tabelaRaca,
    'tabela-exame': tabelaExame,
    'form-especies': formEspecie,
    'form-cliente': formCliente,
    'form-raca': formRaca,
    'form-exame': formExame
  },
  // props: ['tipoCadastro'],
  data() {
    return {
      formVisible: false,
      form: {},
      localDB: '',
      upHere: false,
      tableData: [],
      dataBR: '',
      appURL: 'http://zoom.local/',
      loading: false,
      tipoCadastro: ''
    }
  },
  computed: {

  },
  mounted() {
    this.tipoCadastro = this.$route.params.tipo
    this.configLocalDB()
    this.getDados()
  },
  methods: {
    configLocalDB() {
      this.localDB = localForage.createInstance({
        name: 'cadastros',
        storeName: this.tipoCadastro
      })
      // localForage.config({
      //   name: 'cadastros',
      //   storeName: this.tipoCadastro
      // })
    },
    saveDados: function() {
      const self = this
      var id = this.form.id

      axios.put(self.appURL + self.tipoCadastro + '/' + id, self.form)
        .then(function(response) {
          self.form = {}
          var pathRoute = '/cadastros/index/' + self.tipoCadastro
          self.$router.push({ path: pathRoute })
        })
        .catch(function(error) {
          if (error) {
            self.$alert('Ocorreu um erro ao inserir os dados.', 'Oops', { type: 'error', confirmButtonText: 'OK' })
          }
        })
        .finally(function() {
          self.formVisible = false
          self.getDados()
          self.form = {}
        })

      /*
      self.localDB.setItem(id, this.form).then(function() {
        // location.reload();
        // vm.$router.push({ path: '/clientes' })
        self.formVisible = false
        self.getDados()
        self.form = {}
      })
      */
    },
    removeItem(row) {
      const self = this
      var id = row.id
      var r = confirm('Deseja excluir?')
      if (!r) {
        return false
      }
      self.loading = true
      axios.delete(self.appURL + self.tipoCadastro + '/' + id).then(function(response) {
        self.getDados()
      }).catch(function(error) {
        console.log(error)
      }).finally(function() {
        self.loading = false
      })

      /*
      // return false;
      self.localDB.removeItem(id).then(function() {
        // location.reload();
        // vm.$router.push({ path: '/clientes' })
        self.formVisible = false
        self.getDados()
        self.form = {}
      })
      */
    },
    getForm(row = null, isNew = null) {
      const self = this
      const id = row.row.id
      this.loading = true
      axios.get(this.appURL + this.tipoCadastro + '/' + id).then(function(response) {
        if (isNew) {
          self.form = []
        } else {
          // return console.log(response);
          self.form = response.data
        }
        self.formVisible = true
      }).catch(function(error) {
        console.log(error)
      })
        .finally(() => (this.loading = false))

      /*
      self.localDB.getItem(id).then(function(value) {
        if (isNew) {
          self.form = []
        } else {
          self.form = value
        }
        self.formVisible = true
      }).catch(function(err) {
        console.log('ops', err)
      })
      */
    },
    formatDate: function(row, column, date) {
      return moment(date).format('DD/MM/YYYY')
    },
    getDados: function() {
      // [nome, razao_social, cnpj, cpf, data_nascimento, email]
      console.log('getDados...')
      var self = this
      this.loading = true
      axios.get(this.appURL + this.tipoCadastro).then(function(response) {
        self.tableData = response.data
      }).catch(function(error) {
        console.log(error)
      })
        .finally(() => (this.loading = false))
      /*
      this.tableData = []
      var data = this.tableData
      self.localDB.iterate(function(value, key, iterationNumber) {
        data.push(value)
      }).then(function() {
        console.log('Iteration has completed', data)
        self.loading = false
      }).catch(function(err) {
        // This code runs if there were any errors
        console.log(err)
      })
      */
    }
  }
}
</script>

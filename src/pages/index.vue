<template>
  <v-row>
    <v-col cols="12">
      <v-form ref="formCreateEmployees" 
        v-model="validForm"
        lazy-validation>
        <v-row>
          <div class="d-flex flex-column col-12 col-sm-9 justify-center">          
            <v-text-field v-model="numberEmployeesComputed" 
              type="number" :min="1" :max="50" 
              placeholder="Ingresa un número para crear empleados"
              :rules="rules.numberEmployees">
            </v-text-field>
          </div>
          <div class="d-flex flex-column col-12 col-sm-3 justify-center">
            <v-btn @click="clickButton()" v-if="validForm" color="primary">Create Employees</v-btn>
            <v-btn disabled v-else color="primary">Create Employees</v-btn>
          </div>
        </v-row>
      </v-form>
    </v-col>
    <v-col cols="12">
      <Table :items="items" :headers="headers" :search="search" @changeSearch="changeSearch($event)"></Table>
    </v-col>
  </v-row>
</template>

<script>
import { firestore, doc } from '~/plugins/firebase.js'
import Table from '@/components/Table.vue'
import { addDoc, collection, getDoc, getDocs, limit, onSnapshot, query } from '@firebase/firestore'
export default {
  name: 'IndexPage',
  components: {
    Table
  },
  data () {
    return {
      items: [],
      headers: [
        {
          text: 'Code',
          sortable: true,
          value: 'id',
        },
        {
          text: 'Name',
          sortable: true,
          value: 'name',
        },
        {
          text: 'Salary',
          sortable: true,
          value: 'salary',
        },
      ],
      search: '',
      numberEmployees: 0,
      rules: {
        numberEmployees: [
          v => !!v || 'Number of Employees is required',
          v => v > 0 || 'Dont must be zero'
        ]
      },
      validForm: false
    }
  },
  computed: {
    numberEmployeesComputed: {
      get(){
        return this.numberEmployees
      },
      set(value){
        this.numberEmployees = Number(value)
      }
    }
  },
  methods: {
    changeSearch($event){
      this.search = $event
    },
    async getEmployees(){
      const employeesQuery = query(
        collection(firestore, 'employees')
      )
      onSnapshot(employeesQuery, (querySnapShot) => {
        this.items = querySnapShot.docs.map((e) => {
          return {
            ...e.data(),
            id: e.id
          }
        })
      })
    },
    async clickButton(){
      for (let i = 0; i < this.numberEmployees; i++) {
        await this.createEmployees()
      }
    },
    async createEmployees(){
      let res = await this.$axios.$get('/api/')
      if(res.name){
        const employeesCollection = collection(firestore, 'employees')
        const newDoc = await addDoc(employeesCollection, {
          name: res.name,
          salary: Math.floor(Math.random() * 10000)
        })
      } 
    }
  },
  created(){
    this.getEmployees()
  }
}
</script>

<template>
  <div id="app">
    <nav class="navbar navbar-expand-md navbar-dark bg-dark mb-4">

      <a href="#" class="navbar-brand">Contract Monitor</a>


      <input type="text" class="form-control mr-sm-2" placeholder="Contract Address" v-model="contractAddress">
      <button class="btn btn-outline-success my-2 my-sm-0" @click="getAllContractData">Search</button>
    </nav>

    <div class="card text-white bg-danger mb-3" v-if="showError">
      <div class="card-header">
        <h5 class="card-title">Error</h5>
      </div>
      <div class="card-body">
        <p>Error recieved from API: {{errorMessage}}</p>
      </div>
    </div>

    <hr v-if="showError">

    <div class="card">
      <div class="card-header">
        <h5 class="card-title">Summary ({{contractAddress}})</h5>
      </div>
      <div class="card-body">
        <div class="row">
          <div class="col-sm">
            <ul class="list-group list-group-flush">
              <p v-if="dataLoaded"><b>Balance (Ether)</b>: {{contractDetails.balance / (10**18)}}</p>
              <p v-if="dataLoaded"><b>Created On</b>: {{contractDetails.createdAt}}</p>
              <p v-if="dataLoaded"><b>Last Updated On</b>: {{retrievedAt}}</p>
              <p v-if="dataLoaded"><b>Transaction Types:</b>: {{transactionsDoughnutChartLabels}}</p>
              <p v-if="dataLoaded"><b>Transaction Counts:</b>: {{transactionsDoughnutChartData}}</p>
              <p v-if="dataLoaded"><b>Message Types:</b>: {{messagesDoughnutChartLabels}}</p>
              <p v-if="dataLoaded"><b>Message Counts:</b>: {{messagesDoughnutChartData}}</p>
            </ul>
          </div>
          <div class="col-sm chart-container" v-if="dataLoaded">
            <h5>Transactions</h5>
            <doughnut-chart :chart-data="transactionsDoughnutChartData" :chart-labels="transactionsDoughnutChartLabels"></doughnut-chart>
          </div>
          <div class="col-sm chart-container" v-if="dataLoaded">
            <h5>Messages</h5>
            <doughnut-chart :chart-data="messagesDoughnutChartData" :chart-labels="messagesDoughnutChartLabels"></doughnut-chart>
          </div>
        </div>

      </div>
    </div>

    <hr>

    <div class="card text-center">
      <div class="card-header">
        <ul class="nav nav-tabs card-header-tabs">
          <li class="nav-item">
            <a href="#" class="nav-link" :class="{active: transactionsSelected}" @click="selectTransactions">Transactions</a>
          </li>
          <li class="nav-item">
            <a href="#" class="nav-link" :class="{active: messagesSelected}" @click="selectMessages">Messages</a>
          </li>
        </ul>
      </div>
      <div class="card-body">
        <transactions :transactions="transactions" v-if="dataLoaded && transactionsSelected"></transactions>
        <messages :messages="messages" v-if="dataLoaded && messagesSelected"></messages>
      </div>
    </div>

    <!--<hr>-->

    <!--<div class="card">-->
      <!--<div class="card-header">-->
        <!--<h5 class="card-title">Transactions Metrics</h5>-->
      <!--</div>-->
      <!--<div class="card-body">-->
        <!--<line-chart :chart-data="chartData" v-if="dataLoaded"></line-chart>-->
      <!--</div>-->
    <!--</div>-->
  </div>
</template>

<script>
  import axios from 'axios'
  import Transactions from './components/Transactions'
  import Messages from './components/Messages'
  import LineChart from './components/LineChart'
  import DoughnutChart from './components/DoughnutChart'

  export default {
    components: {
      Transactions,
      Messages,
      LineChart,
      DoughnutChart
    },
    data() {
      return {
        contractAddress: '',
        transactionsSelected: true,
        messagesSelected: false,
        contractDetails: {
          balance: 0,
          createdAtTimestamp: 0,
          createdAt: null,
        },
        transactions: [],
        messages: [],
        transactionsDoughnutChartData: [],
        transactionsDoughnutChartLabels: [],
        messagesDoughnutChartLabels: [],
        messagesDoughnutChartData: [],
        lineChartData: [],
        polling: null,
        dataLoaded: false,
        retrievedAt: null,
        errorMessage: '',
        showError: false
      }
    },
    methods: {
      getAllContractData() {
        this.getContractDetails()
        this.getContractTransactionsDetails()
        this.getContractMessages()
        this.retrievedAt = new Date()
        this.dataLoaded = true
        this.pollData()
      },
      getContractTransactionsDetails() {
        let url = 'https://api.aleth.io/v1/contracts/' + this.contractAddress + '/transactions';

        axios.get(url)
          .then(response => {
            this.transactions = response.data.data

            this.transactionsDoughnutChartLabels = [...new Set(this.transactions.map(txn => txn.attributes.msgPayload.funcName))]

            this.transactionsDoughnutChartData = Object.values(this.transactions.reduce((acc, txn) => {
              let funcName = txn.attributes.msgPayload.funcName
              acc[funcName] = acc[funcName] ? acc[funcName] + 1 : 1
              return acc
            }, Object.create(null)))

          }).catch(err => {
            console.log('unable to get transactions' + err)
          })
      },
      getContractDetails() {
        let url = "https://api.aleth.io/v1/contracts/" + this.contractAddress;

        axios.get(url)
          .then(response => {
            let results = response.data.data

            this.contractDetails.balance = results.attributes.balance
            this.contractDetails.createdAtTimestamp = results.attributes.createdAtTimestamp
            this.contractDetails.createdAt = new Date(this.contractDetails.createdAtTimestamp * 1000)
          }).catch(err => {
            this.errorMessage = err.response.data.errors[0].title;
            this.showError = true
          })
      },
      getContractMessages() {
        let url = 'https://api.aleth.io/v1/contracts/' + this.contractAddress + '/contractMessages';

        axios.get(url)
          .then(response => {
            this.messages = response.data.data

            this.messagesDoughnutChartLabels = [...new Set(this.messages.map(msg => msg.attributes.msgType))]

            this.messagesDoughnutChartData = Object.values(this.messages.reduce((acc, msg) => {
              let msgType = msg.attributes.msgType
              acc[msgType] = acc[msgType] ? acc[msgType] + 1 : 1
              return acc
            }, Object.create(null)))
          }).catch(err => {
          console.log('unable to get messages' + err)
        })
      },
      selectTransactions() {
        this.transactionsSelected = true
        this.messagesSelected = false
      },
      selectMessages() {
        this.transactionsSelected = false
        this.messagesSelected = true
      },
      pollData() {
        this.polling = setInterval(() => {
          this.getAllContractData()
        }, 60 * 1000)
      }
    },
    mounted() {
      if (this.contractAddress !== '') {
        this.getContractDetails()
      }
    }
  }
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
}

.chart-container {
  text-align: center;
}
</style>
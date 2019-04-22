<script>
  import { Line, mixins } from 'vue-chartjs'
  const { reactiveProp } = mixins

  export default {
    extends: Line,
    mixins: [reactiveProp],
    props: {
      chartData: {
        type: Array | Object,
        required: true
      }
    },
    data() {
      return {
        options: {
          scales: {
            yAxes: [{
              ticks: {
                beginAtZero: false
              },
              gridLines: {
                display: true
              }
            }],
            xAxes: [ {
              type: 'time',
              gridLines: {
                display: false
              }
            }]
          },
          legend: {
            display: true
          },
          responsive: true,
          maintainAspectRatio: false
        }
      }
    },
    mounted () {
      this.renderChart({
        datasets: [
          {
            label: 'transactions',
            borderColor: '#249EBF',
            pointBackgroundColor: 'white',
            borderWidth: 1,
            pointBorderColor: '#249EBF',
            backgroundColor: 'transparent',
            data: this.chartData
          }
        ]
      }, this.options)
    }
  }
</script>

<style scoped>

</style>

<script>
import { Line } from 'vue-chartjs';

export default {
  name: 'DealsLineChart',
  extends: Line,
  props:{
    chartData: {
      required: true
    },
    options: {
      type: Object
    }
  },
  mounted () {
    this.renderChart(this.chartData, this.options)
  }
};
</script>

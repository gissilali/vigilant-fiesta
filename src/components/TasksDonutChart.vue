<script>
import { Doughnut } from 'vue-chartjs';

const plugin = function (chart) {
  const width = chart.chart.width;
  const height = chart.chart.height;
  const ctx = chart.chart.ctx;

  ctx.restore();
  const fontSize = (height / 114).toFixed(1.4);
  ctx.font = fontSize + 'em sans-serif';
  ctx.textBaseline = 'middle';

  const text = "60%";
  const textX = Math.round((width - ctx.measureText(text).width) / 2);
  const textY = height / 1.75;

  ctx.fillText(text, textX, textY);
  ctx.save();
};
export default {
  extends: Doughnut,
  name: 'TasksDonutChart',
  props:{
    chartData: {
      required: true
    },
    options: {
      type: Object
    }
  },
  methods: {

  },
  mounted () {
    this.addPlugin({
      id: 'my-plugin',
      beforeDraw: plugin
    })
    this.renderChart(this.chartData, this.options)
  }
};
</script>

<style scoped>

</style>

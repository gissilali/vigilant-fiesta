<script>
import { Doughnut } from 'vue-chartjs';

export default {
  extends: Doughnut,
  name: 'TasksDonutChart',
  props:{
    chartData: {
      required: true
    },
    options: {
      type: Object
    },
    completedTasks: {
      required: false
    }
  },
  methods: {
    /**
     * Plugin to display the text in the middle of the doughnut chart
     * @param chart
     */
    textDisplayPlugin (chart) {
      const width = chart.chart.width;
      const height = chart.chart.height;
      const ctx = chart.chart.ctx;

      ctx.restore();
      const fontSize = (height / 114).toFixed(1.4);
      ctx.font = fontSize + 'em sans-serif';
      ctx.textBaseline = 'middle';

      const text = `${this.completedTasks}%`;
      const textX = Math.round((width - ctx.measureText(text).width) / 2);
      const textY = height / 1.75;

      ctx.fillText(text, textX, textY);
      ctx.save();
    }
  },
  mounted () {
    this.addPlugin({
      id: 'my-plugin',
      beforeDraw: this.textDisplayPlugin
    })
    this.renderChart(this.chartData, this.options)
  }
};
</script>

<style scoped>

</style>

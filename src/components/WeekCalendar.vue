<template>
    <div></div>
</template>

<script>
const jsCalendar = require('js-calendar');
const jsCal = new jsCalendar.Generator({ onlyDays: true });

export default {
  name: 'WeekCalendar',
  created() {
    this.thisWeeksDays = jsCal(2021, 1, jsCalendar.addLabels).cells
    console.log(this.thisWeeksDays)
  },
  data() {
    return {
      thisWeeksDays: []
    };
  },
};
</script>

<style scoped>

</style>

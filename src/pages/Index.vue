<template>
  <q-page class="q-pa-md-xl q-pa-xs-xs bg-gray">
    <div class="row q-col-gutter-xl q-col-gutter-xs-md">
      <div class="col-md-8 col-xs-12">
        <q-card flat>
          <q-card-section>
            <p>8 Tasks Completed out of 10</p>
            <q-linear-progress :value="0.8" color="positive" class="q-mt-md"/>
            <div>
              <week-calendar/>
            </div>
          </q-card-section>
          <q-card-section>
            <q-card>
              <q-card-section>
                <div class="row">
                  <div class="col">
                    <p class="text-weight-medium text-grey-9 q-mb-xs text-h6">Send review by Monday</p>
                    <span class="text-grey-7"><span>Due Date:</span> 5th March 2021</span>
                  </div>
                  <span class="text-grey-7 q-mt-xs text-weight-medium">Reminder</span>
                </div>
                <div class="row q-mt-md">
                  <div class="col flex items-center">
                    <q-avatar size="40px" class="q-mr-md" color="orange">J</q-avatar>
                    <span class="text-grey-7 ">Joseph K. Link</span>
                  </div>
                  <span class="text-grey-7 flex items-center q-mt-xs text-weight-medium">
                     <q-badge color="positive">Completed</q-badge>
                  </span>
                </div>
              </q-card-section>
            </q-card>
          </q-card-section>
        </q-card>
      </div>
      <div class="col-md-4 col-xs-12">
        <q-card flat>
          <q-card-section>
            <p class="text-weight-medium text-grey-9 q-mb-xs text-h6">Deals</p>
            <deals-line-chart :chart-data="dealsChartData"></deals-line-chart>
          </q-card-section>
        </q-card>
        <q-card class="q-mt-lg" flat>
          <q-card-section>
            <p class="text-weight-medium text-grey-9 q-mb-xs text-h6">Deals</p>
            <tasks-donut-chart :options="donutChartOptions" :chart-data="tasksCompletionData"/>
          </q-card-section>
        </q-card>
      </div>
    </div>
  </q-page>
</template>

<script>
import WeekCalendar from 'components/WeekCalendar';
import DealsLineChart from 'components/DealsLineChart';
import TasksDonutChart from 'components/TasksDonutChart';

export default {
  name: 'PageIndex',
  components: {
    TasksDonutChart,
    DealsLineChart,
    WeekCalendar
  },
  data() {
    return {
      donutChartOptions: {
        responsive: true,
        legend: {
          position: 'top',
        },
        borderWidth: 5,
        title: {
          display: true,
          text: 'Chart.js Doughnut Chart'
        },
        animation: {
          animateScale: true,
          animateRotate: true
        }
      },
      dealsChartData: {
        labels: [this.getRandomInt(), this.getRandomInt(), this.getRandomInt(), this.getRandomInt(), this.getRandomInt(), this.getRandomInt()],
        datasets: [
          {
            label: 'Closed Deals',
            backgroundColor: '#31CCEC',
            data: [
              this.getRandomInt(), this.getRandomInt(), this.getRandomInt(), this.getRandomInt(), this.getRandomInt(), this.getRandomInt(), this.getRandomInt()]
          }
        ]
      },
      tasksCompletionData: {
        datasets: [{
          data: [
            60,
            20,
            30,
          ],

          backgroundColor: [
            '#2ed47a',
            '#ffb946',
            '#f7685b',
          ],
          label: 'Tasks'
        }],
        labels: [
          'Complete',
          'Incomplete',
          'Ended'
        ]
      }
    };
  },
  methods: {
    getRandomInt() {
      return Math.floor(Math.random() * (50 - 5 + 1)) + 5;
    }
  }
};
</script>

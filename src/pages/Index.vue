<template>
  <q-page class="q-pa-md-xl q-pa-xs-xs bg-gray">
    <div class="row q-col-gutter-md-xl q-col-gutter-xs-xs">
      <div class="col-md-8 col-xs-12">
        <q-card class="q-pa-md-lg q-pa-xs-none" flat>
          <q-card-section>
            <p>8 Tasks Completed out of 10</p>
            <q-linear-progress :value="0.8" color="positive" class="q-mt-md"/>
            <div>
              <week-calendar/>
            </div>
          </q-card-section>
          <q-card-section>
            <dashboard-task-item class="q-mb-lg" v-for="(task, index) in recentTasks" :key="index" :task="task"/>
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
            <tasks-donut-chart :completed-tasks="completeTasksPercentage"  :options="donutChartOptions" :chart-data="tasksCompletionData"/>
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
import axios from 'axios';
import { env } from 'src/env.config';
import DashboardTaskItem from 'components/DashboardTaskItem';
import moment from 'moment';

export default {
  name: 'PageIndex',
  components: {
    DashboardTaskItem,
    TasksDonutChart,
    DealsLineChart,
    WeekCalendar
  },
  created() {
    this.getTasks();
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
          text: 'Task Completion'
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
      tasks: []
    };
  },
  methods: {
    getRandomInt() {
      return Math.floor(Math.random() * (50 - 5 + 1)) + 5;
    },
    async getTasks() {
      const {
        data,
        status
      } = await axios.get(`${env.API_URL}/tasks`);
      if (status === 200) {
        this.tasks = data.data;
      }

      return data.data;
    },
    getTaskStatus(task) {
      /*
        Checking if the task is completed
       */
      if (task.completed_on) {
        return {
          'color': 'positive',
          'description' : 'completed'
        }
      }
      /*
      Checking if the task is past due date and is not completed - assign the ended status
     */
      if(moment(task.due_on).isBefore()  && task.completed_on == null) {
        return {
          'color': 'negative',
          'description' : 'ended'
        }
      }
      /*
           Checking if the task is not yet past due date and is not completed - assign the active status
          */
      if(moment(task.due_on).isAfter() && task.completed_on == null) {
        return {
          'color': 'warning',
          'description' : 'active'
        }
      }
    },
    groupBy(items, key) {
      return items.reduce(function(rv, x) {
        (rv[x[key]] = rv[x[key]] || []).push(x);
        return rv;
      }, {});
    }
  },
  computed: {
    recentTasks() {
      return [...this.tasks].slice(0, 3)
    },
    tasksDataSet() {
      /**
       * Group a clone of the tasks by the status description,
       * this way we can count tasks by their completion statuses
       * for our graph data
       */
      return this.groupBy([...this.tasks].map((task) => {
        return this.getTaskStatus(task)
      }), 'description')
    },
    tasksCompletionData() {
      return {
        datasets: [{
          data: [
            this.tasksDataSet.completed ? this.tasksDataSet.completed.length : 0,
            this.tasksDataSet.active ? this.tasksDataSet.active.length : 0,
            this.tasksDataSet.ended ? this.tasksDataSet.ended.length : 0,
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
    },
    completeTasksPercentage() {
      return Math.round((this.tasksDataSet.completed.length /this.tasks.length) * 100);
    }
  }
};
</script>

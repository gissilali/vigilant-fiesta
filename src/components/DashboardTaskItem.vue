<template>
  <q-card>
    <q-card-section>
      <div class="row">
        <div class="col">
          <p class="text-weight-medium text-grey-9 q-mb-xs text-h6">{{ task.description }}</p>
          <span class="text-grey-7"><span>Due Date:</span> {{ task.due_on }}</span>
        </div>
        <span class="text-grey-7 q-mt-xs text-weight-medium text-capitalize">{{ task.type }}</span>
      </div>
      <div class="row q-mt-md">
        <div class="col flex items-center">
          <q-avatar size="40px" class="q-mr-md" color="orange">{{ task.assigned_to[0] }}</q-avatar>
          <span class="text-grey-7 ">{{ task.assigned_to }}</span>
        </div>
        <span class="text-grey-7 flex items-center q-mt-xs text-weight-medium">
                     <q-badge :color="status.color">{{ status.description }}</q-badge>
                  </span>
      </div>
    </q-card-section>
  </q-card>
</template>

<script>
import moment from "moment"
export default {
  name: 'DashboardTaskItem',
  props: {
    task: {
      required: true,
      type: Object
    }
  },
  computed: {
    status() {
      if (this.task.completed_on) {
        return {
          'color': 'positive',
          'description' : 'completed'
        }
      }
      if(moment(this.task.due_on).isBefore()  && this.task.completed_on == null) {
        return {
          'color': 'negative',
          'description' : 'ended'
        }
      }

      if(moment(this.task.due_on).isAfter() && this.task.completed_on == null) {
        return {
          'color': 'warning',
          'description' : 'active'
        }
      }
    },

  }
};
</script>

<style scoped>

</style>

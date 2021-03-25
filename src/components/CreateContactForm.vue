<template>
  <form @submit.prevent="submit()" :action="action">
    <div class="q-mb-lg">
      <q-input
        :error="this.form.errors.hasOwnProperty('name')"
        :error-message="`${this.form.errors.hasOwnProperty('name') ? this.form.errors.name[0] : '' }`"
        label="Contact Name" autofocus outlined id="name" name="name" v-model="form.name"/>
    </div>
    <div class="q-mb-lg">
      <q-input
        :error="this.form.errors.hasOwnProperty('email')"
        :error-message="`${this.form.errors.hasOwnProperty('email') ? this.form.errors.email[0] : '' }`"
        label="Contact Email" outlined type="email" id="email" name="email" v-model="form.email"/>
    </div>
    <div class="q-mb-lg">
      <q-select
        :error="this.form.errors.hasOwnProperty('company')"
        :error-message="`${this.form.errors.hasOwnProperty('company') ? this.form.errors.company[0] : '' }`"
        filled
        v-model="form.company"
        use-input
        map-options
        emit-value
        :multiple="false"
        input-debounce="0"
        :options="companyOptions"
        @filter="filterFn"
      />
    </div>
    <div class="q-mb-lg">
      <q-select
        :error="this.form.errors.hasOwnProperty('role')"
        :error-message="`${this.form.errors.hasOwnProperty('role') ? this.form.errors.role[0] : '' }`"
        outlined v-model="form.role" :options="roleOptions" label="Role"/>
    </div>
    <div class="q-mb-lg">
      <q-file
        :error="this.form.errors.hasOwnProperty('avatar')"
        :error-message="`${this.form.errors.hasOwnProperty('avatar') ? this.form.errors.avatar[0] : '' }`"
        accept="image/png,image/jpeg,image/jpg" outlined v-model="form.avatar" label="Upload Avatar Image">
        <template v-slot:prepend>
          <q-icon name="image"/>
        </template>
      </q-file>
    </div>
    <div class="q-mb-lg relative-position q-px-sm">
      <q-item-label>Forecast</q-item-label>
      <div class="q-py-lg">
        <q-slider class="q-mb-lg" label :label-value="form.forecast + '%'" v-model="form.forecast" :min="0" :max="100"/>
      </div>
      <div v-if="this.form.errors.hasOwnProperty('forecast')"
           class="q-field__bottom q-mb-lg q-mt-md row items-start q-field__bottom--animated">
        <div class="q-field__messages col">
          <div class="text-negative ">{{ this.form.errors.forecast[0] }}</div>
        </div>
      </div>
    </div>
    <div class="q-mb-lg">
      <q-btn :loading="formSubmitting" size="lg" label="Submit" type="submit" color="primary" class="full-width"/>
    </div>
  </form>
</template>

<script>
import axios from 'axios';
import Fuse from 'fuse.js'
import {env} from 'src/env.config';

export default {
  name: 'CreateContactForm',
  props: {
    companies: {
      required: true,
      type: Array
    }
  },
  created() {
    axios.get(`${env.API_URL}/roles`)
      .then(({
        data,
        status,
      }) => {
        if (status === 200) {
          this.roleOptions = data.data.map(function (role) {
            return {
              label: role.name,
              value: role.id
            };
          });
        }
      })
      .catch(({ response }) => {
        console.log({ response });
      });



    this.$nextTick(() => {
      this.companyOptions = this.companies.map((company) => {
        return {
          value: company.id,
          label: company.name
        };
      });

      if (this.fuseSearch == null) {
        const options = {
          includeScore: true,
          keys: ['label'],
          threshold: 0.5
        }

        this.fuseSearch = new Fuse([...this.companyOptions], options)
      }
    })
  },
  data() {
    return {
      companyOptions: [],
      form: {
        avatar: null,
        name: '',
        email: '',
        company: '',
        role: '',
        forecast: 0,
        errors: {}
      },
      formSubmitting: false,
      roleOptions: [],
      action: `${env.API_URL}/contacts/store`,
      fuseSearch: null
    };
  },
  methods: {
    async submit() {
      this.formSubmitting = true;
      try {
        const formData = new FormData;
        formData.append('avatar', this.form.avatar);
        formData.append('name', this.form.name);
        formData.append('company', this.form.company);
        formData.append('forecast', this.form.forecast);
        formData.append('email', this.form.email);
        formData.append('role', this.form.role.value);
        const {
          data,
          status
        } = await axios.post(`${env.API_URL}/contacts/store`, formData);
        if (status === 201) {
          this.$emit('event-created', data.data);
        }
      } catch ({ response }) {
        this.formSubmitting = false;
        /**
         * Catching validation errors from our backend
         */
        if (response.status === 422) {
          this.form.errors = response.data.errors;
          return;
        }
        /*
          Generic error message for what would likely be a server error in our backend
         */
        this.$q.notify({
          message: 'failed to add contacts',
          color: 'negative'
        });
      }
    },
    /*
      We use a text search function to filter through our
      list of companies populating the select input
      using the fuse.js fulltext search npm package
     */
    filterFn(searchString, update) {
      if (searchString === '') {
        update(() => {
          this.companyOptions = this.companies.map((company) => {
            return {
              value: company.id,
              label: company.name
            };
          });
        });
        return;
      }

      update(() => {
        const  results = this.fuseSearch.search(searchString);
        this.companyOptions = results.map((result) => result.item)
      })
    }
  },
};
</script>

<style scoped>

</style>

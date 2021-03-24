<template>
  <q-page class="q-pa-xl">
    <div style="align-items: center" class="flex justify-between">
      <q-btn-dropdown
        unelevated
        :outline="false"
        class="btn shadow-0 text-primary btn-link"
        @click="onMainClick"
      >
        <template v-slot:label>
          <div class="row items-center shadow-0 no-wrap">
            <div class="text-center text-grey-9 q-btn--no-uppercase">
              Companies
            </div>
          </div>
        </template>
        <q-list>
          <q-item clickable v-close-popup @click="onItemClick(null)">
            <q-item-section>
              <q-item-label>All Companies</q-item-label>
            </q-item-section>
          </q-item>
          <q-item v-for="(company, index) in companies" :key="index"
                  clickable v-close-popup @click="onItemClick(company.id)">
            <q-item-section>
              <q-item-label>{{ company.name }}</q-item-label>
            </q-item-section>
          </q-item>
        </q-list>
      </q-btn-dropdown>
      <q-btn color="primary" size="lg"
             @click.prevent="addContact = true" class="q-btn--no-uppercase"
             label="Add Contact"/>
    </div>
    <div class="q-mt-xl">
      <q-table
        title="Contacts"
        :data="data"
        :columns="columns"
        row-key="id"
        :selected-rows-label="getSelectedString"
        selection="multiple"
        :loading="loading"
        :selected.sync="selected"
      >

        <template v-if="selectedIds.length > 0" v-slot:top>
          <div class="flex items-center">
            <span class="q-mr-lg">{{ selectedIds.length }} selected</span>
            <span class="flex justify-between items-center">
              <q-btn class="text-dark q-mr-lg"
                     flat
                     dense
                     round
                     size="sm"
                     icon="delete"
                     aria-label="Delete" @click.prevent="attemptDeletion" :disable="loading"/>
            </span>
          </div>
        </template>
        <template v-slot:loading>
          <q-inner-loading showing color="primary"/>
        </template>
        <template v-slot:body-cell-contact="props">
          <q-td :props="props">
            <div class="flex items-center">
              <div class="avatar q-mr-md" :style="{
                'background-image': 'url('+ props.row.contact.avatar + ')',
                'background-size': 'cover',
              }">

              </div>
              <span class="text-weight-bold">{{ props.row.contact.name }}</span>
            </div>
          </q-td>
        </template>
        <template v-slot:body-cell-recent_activity="props">
          <q-td :props="props">
            <span>{{ props.value.displayString }}</span>
          </q-td>
        </template>
      </q-table>
    </div>
    <q-dialog size="md" v-model="addContact">
      <q-card style="width: 500px; max-width: 80vw;" bordered>
        <q-card-section>
          <div class="q-my-0 text-h6">Create Contact</div>
        </q-card-section>

        <q-separator inset/>

        <q-card-section>

          <create-contact-form :companies="companies" @event-created="addContactToList"/>

        </q-card-section>
      </q-card>

    </q-dialog>
  </q-page>
</template>

<script>
import axios from 'axios';
import moment from 'moment';
import CreateContactForm from 'components/CreateContactForm';
import {env} from 'src/env.config';

export default {
  name: 'List',
  components: { CreateContactForm },
  async created() {
    this.getContacts();
    this.getCompanies();
  },
  data() {
    return {
      addContact: false,
      selected: [],
      companies: [],
      columns: [
        {
          name: 'contact',
          required: true,
          label: 'Name',
          align: 'left',
          field: (row) => row.name,
          format: (val) => `${val}`,
          sortable: true,
        },
        {
          name: 'email',
          align: 'left',
          label: 'Email',
          field: 'email',
          sortable: false,
        },
        {
          name: 'company',
          label: 'Company name',
          field: 'company',
          align: 'left',
          sortable: true,
        },
        {
          name: 'role',
          label: 'Role',
          field: 'role',
          align: 'left',
        },
        {
          name: 'forecast',
          label: 'Forecast',
          field: 'forecast',
          align: 'right',
          sort: (a, b) => parseInt(a, 10) - parseInt(b, 10),
          sortable: true,
        },
        {
          name: 'recent_activity',
          label: 'Recent Activity',
          field: 'recent_activity',
          align: 'left',
          sortable: true,
          sort: (a, b) => parseInt(b.minutesFromNow, 10) - parseInt(a.minutesFromNow, 10),
        },
      ],
      contacts: [],
      filteredContacts: [],
      loading: false,
    };
  },
  computed: {
    data() {
      return this.filteredContacts.map((contact) => {
        const recentActivity = {
          displayString: moment(contact.last_activity)
            .fromNow(),
          minutesFromNow: moment()
            .diff(moment(contact.last_activity)),
        };
        return {
          id: contact.id,
          contact: {
            name: contact.name,
            avatar: contact.avatar,
          },
          email: contact.email,
          company: contact.company.name,
          role: contact.role.name,
          forecast: `${contact.forecast}%`,
          recent_activity: recentActivity,
        };
      });
    },
    selectedIds() {
      return this.selected.map((contact) => contact.id);
    },
  },
  methods: {
    onMainClick() {
      if (this.companies.length === 0) {
        this.getCompanies();
      }
    },
    async getContacts() {
      this.loading = true;
      try {
        const {
          data,
          status,
        } = await axios.get(`${env.API_URL}/contacts`);
        if (status === 200) {
          this.loading = false;
          this.contacts = data.data;
          this.filteredContacts = this.contacts;
        }
      } catch (e) {
        this.loading = false;
        this.$q.notify({
          message: 'Error fetching contacts',
          color: 'red',
        });
      }
    },
    onItemClick(companyId) {
      if (companyId == null) {
        this.filteredContacts = [...this.contacts];
        return;
      }
      const copyOfContacts = [...this.contacts];
      this.filteredContacts = copyOfContacts.filter((contact) => contact.company.id === companyId);
    },
    getSelectedString() {
      return this.selected.length === 0 ? '' : `${this.selected.length} record${this.selected.length > 1 ? 's' : ''} selected of ${this.data.length}`;
    },
    async getCompanies() {
      try {
        const {
          data,
          status,
        } = await axios.get(`${env.API_URL}/companies`);
        if (status === 200) {
          this.companies = data.data;
        }
      } catch (e) {
        this.$q.notify({
          message: 'Error fetching companies',
          color: 'red',
        });
      }
    },
    attemptDeletion() {
      this.$q.dialog({
        title: 'Confirm',
        message: `You are about to delete ${this.selectedIds.length} contact${this.selectedIds.length !== 1 ? 's' : ''}`,
        cancel: true,
        persistent: true,
      })
        .onOk(() => {
          axios.post(`${env.API_URL}/contacts/delete`, {
            contacts: this.selectedIds,
          })
            .then(({
              status,
              message,
            }) => {
              if (status === 200) {
                this.$q.notify({
                  message: 'Contacts Deleted',
                  caption: message,
                  color: 'positive',
                });
                this.removeDeleted(this.selectedIds);
                this.selected = [];
              }
            })
            .catch(({ response }) => {
              if (response.status === 422) {
                this.$q.notify({
                  message: response.data.errors.contacts[0],
                  color: 'negative',
                });
                return;
              }
              this.$q.notify({
                message: 'Failed to delete',
                color: 'negative',
              });
            });
        })
        .onOk(() => {
          // console.log('>>>> second OK catcher')
        })
        .onCancel(() => {
          // console.log('>>>> Cancel')
        })
        .onDismiss(() => {
          // console.log('I am triggered on both OK and Cancel')
        });
    },
    removeDeleted(selectedIds) {
      this.filteredContacts = [
        ...this.contacts.filter((contact) => !selectedIds.includes(contact.id)),
      ];
    },
    addContactToList(contact) {
      console.log({contact})
      this.addContact = false;
      // this.getContacts()
      this.filteredContacts.unshift(contact)
    }
  },
};
</script>

<style scoped>

</style>

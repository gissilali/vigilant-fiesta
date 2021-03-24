<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar class="q-pa-lg flex justify-between bg-white">
        <div class="flex" style="align-items: center">
          <span>
          <q-btn
            class="text-dark q-mr-lg"
            flat
            dense
            round
            icon="menu"
            aria-label="Notification"
            @click="toggleDrawer()"
          />
          </span>
          <q-input v-model="contactSearchQuery">
            <template v-slot:prepend>
              <q-icon name="search"/>
            </template>
          </q-input>
        </div>
        <div>
          <q-btn
            class="text-dark"
            flat
            dense
            round
            icon="notifications_none"
            aria-label="Notification"
          />
        </div>
      </q-toolbar>
    </q-header>

    <q-drawer
      v-model="drawerOpen"
      :mini="miniDrawerOpen"
      show-if-above
      bordered
      @hide="drawerIsOpen = false"
      content-class="bg-grey-1"
    >
      <q-list>
        <q-item-label
          header
          class="text-grey-8 q-pa-none"
        >
          <div class="q-pa-lg flex justify-between" style="border-bottom: 1px solid #ebeff2">
            <h5 class="q-my-md text-primary text-weight-bold">Sass Kit</h5>
          </div>

        </q-item-label>
        <q-item
          v-show="!drawerOpen"
          :clickable="false"
          tag="div"
          class="q-py-lg"
        >
          <h6 class="q-my-md  text-weight-bold text-primary">Sass</h6>
        </q-item>
        <q-item
          :clickable="false"
          tag="div"
          class="q-py-lg"
        >
          <div class="avatar">
            <span>S</span>
          </div>
          <q-item-section class="q-ml-md">
            <q-item-label>Gibson Silali</q-item-label>
            <q-item-label caption>
              gibson.silali@gmail.com
            </q-item-label>
          </q-item-section>
        </q-item>
        <EssentialLink
          v-for="link in essentialLinks"
          :key="link.title"
          v-bind="link"
        />
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view/>
    </q-page-container>
  </q-layout>
</template>

<script>
import EssentialLink from 'components/EssentialLink.vue';

const linksData = [
  {
    title: 'Dashboard',
    icon: 'dashboard',
    link: '/',
  },
  {
    title: 'Tasks',
    icon: 'list',
    link: '/tasks',
  },
  {
    title: 'Email',
    icon: 'email',
    link: '/emails',
  },
  {
    title: 'Contacts',
    icon: 'account_circle',
    link: '/contacts',
  },
  {
    title: 'Chat',
    icon: 'chat',
    link: '/chat',
  },
  {
    title: 'Deals',
    icon: 'view_array',
    link: '/deals',
  },
];

export default {
  name: 'MainLayout',
  components: { EssentialLink },
  created() {
      this.drawerIsOpen = false;
  },
  data() {
    return {
      drawerOpen: false,
      miniDrawerOpen: false,
      drawerIsOpen: true,
      essentialLinks: linksData,
      contactSearchQuery: '',
    };
  },
  watch: {
    drawerIsOpen: {
      handler: function (drawerIsOpen) {
          if (!this.isMobile()) {
            this.drawerOpen = true;
            this.miniDrawerOpen = !!drawerIsOpen;
          } else {
            console.log({drawerIsOpen})
            this.miniDrawerOpen = false
            this.drawerOpen = drawerIsOpen
          }
      },
      immediate: true
    }
  },
  methods: {
    isMobile() {
      return /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent);
    },
    toggleDrawer() {
      this.drawerIsOpen = !this.drawerIsOpen;
    }
  }
};
</script>

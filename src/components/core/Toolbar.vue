<template>
  <v-toolbar
    id="core-toolbar"
    flat
    prominent
    style="background: #eee;"
  >
    <div class="v-toolbar-title">
      <v-toolbar-title class="tertiary--text font-weight-light truncate">
        <v-btn
          v-if="responsive"
          class="default v-btn--simple"
          dark
          icon
          @click.stop="onClickBtn"
        >
          <v-icon>mdi-view-list</v-icon>
        </v-btn>
        {{ title }}
      </v-toolbar-title>
    </div>

    <v-spacer/>
    <v-toolbar-items v-if="!responsive">
      <v-flex
        align-center
        layout
        py-2
      >
        <router-link
          v-ripple
          v-if="isAuthPresent"
          class="toolbar-items"
          to="/members/newmember"
        >
          <v-icon color="tertiary">mdi-account-plus</v-icon>
        </router-link>

        <router-link
          v-ripple
          v-if="isAuthPresent"
          class="toolbar-items"
          to="/events/newevent"
        >
          <v-icon color="tertiary">mdi-calendar-plus</v-icon>
        </router-link>

        <router-link
          v-ripple
          class="toolbar-items"
          to="/"
        >
          <v-icon color="tertiary">mdi-view-dashboard</v-icon>
        </router-link>

        <v-menu
          v-if="isAuthPresent"
          bottom
          left
          content-class="dropdown-menu"
          offset-y
          transition="slide-y-transition"
        >
          <v-btn
            v-ripple
            slot="activator"
            flat
            class="toolbar-items"
            icon
          >
            <v-icon color="tertiary">mdi-account</v-icon>
          </v-btn>
          <v-card>
            <v-list dense>
              <v-list-tile to="/user-profile">
                <v-list-tile-title>Perfil</v-list-tile-title>
              </v-list-tile>
              <v-list-tile @click="logout">
                <v-list-tile-title>Salir</v-list-tile-title>
              </v-list-tile>
            </v-list>
          </v-card>
        </v-menu>
      </v-flex>
    </v-toolbar-items>
  </v-toolbar>
</template>

<script>
import { mapMutations } from 'vuex'

export default {
  data: () => ({
    title: null,
    responsive: false,
    responsiveInput: false
  }),
  computed: {
    isAuthPresent() {
      return this.$store.getters.isAuth
    }
  },

  watch: {
    $route(val) {
      this.title = val.name
    }
  },

  mounted() {
    this.onResponsiveInverted()
    window.addEventListener('resize', this.onResponsiveInverted)
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.onResponsiveInverted)
  },

  methods: {
    ...mapMutations('app', ['setDrawer', 'toggleDrawer']),
    onClickBtn() {
      this.setDrawer(!this.$store.state.app.drawer)
    },
    onClick() {
      //
    },
    onResponsiveInverted() {
      if (window.innerWidth < 991) {
        this.responsive = true
        this.responsiveInput = false
      } else {
        this.responsive = false
        this.responsiveInput = true
      }
    },
    logout() {
      this.$store.dispatch('logout')
      this.$router.push('/login')
    }
  }
}
</script>

<style>
#core-toolbar a {
  text-decoration: none;
}

.truncate {
  width: 190px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
</style>

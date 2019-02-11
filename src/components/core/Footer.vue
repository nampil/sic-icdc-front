<template>
  <v-footer
    id="core-footer"
    :fixed="responsive"
    height="auto"
    color="secondary"
  >
    <v-layout
      row
      wrap
      justify-space-between
      v-if="responsive"
    >
      <router-link
        v-ripple
        v-if="isAuthPresent"
        to="/members/newmember"
        class="mx-3"
      >
        <v-icon color="white">mdi-account-plus</v-icon>
      </router-link>

      <router-link
        v-ripple
        v-if="isAuthPresent"
        to="/events/newevent"
        class="mx-3"
      >
        <v-icon color="white">mdi-calendar-plus</v-icon>
      </router-link>

      <router-link
        v-ripple
        to="/"
        class="mx-3"
      >
        <v-icon color="white">mdi-view-dashboard</v-icon>
      </router-link>

      <v-menu
        class="mx-3"
        v-if="isAuthPresent"
        top
        center
        content-class="dropdown-menu"
        offset-y
        transition="slide-y-transition"
      >
        <v-btn
          slot="activator"
          icon
          class="ma-0 icon-menu"
        >
          <v-icon color="white">mdi-account</v-icon>
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
    </v-layout>
  </v-footer>
</template>

<script>
export default {
  data: () => ({
    responsive: false
  }),
  computed: {
    isAuthPresent() {
      return this.$store.getters.isAuth
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

<style lang="scss">
#core-footer {
  z-index: 0;
}
.icon-menu.v-btn.v-btn--icon {
  width: 24px;
  height: 24px;
  font-size: 24px;
  margin: 0 16px;
  .v-btn__content {
    i.v-icon.mdi.mdi-account.theme--light.white--text {
      font-size: 24px;
    }
  }
}
</style>

<template>
  <v-container
    v-if="event"
    class="admin-event"
    fill-height
    fluid
    grid-list-xl
  >
    <v-layout wrap>
      <v-toolbar color="info">
        <v-toolbar-title
          v-if="event"
          class="white--text"
        >{{ event.title }}</v-toolbar-title>
        <v-spacer/>
        <v-toolbar-items class="align-center">
          <v-dialog
            v-model="registerGuestModal"
            max-width="600px"
          >
            <v-btn
              slot="activator"
              small
              color="secondary"
              @click="registerGuestModal= true"
            >Registrar Invitado
              <v-icon class="ml-3">mdi-account-card-details</v-icon>
            </v-btn>
            <v-card>
              <v-card-text>
                <guests-RegisterGuests
                  :event-id="event._id"
                  @close-modal="closeModal"
                />
              </v-card-text>
            </v-card>
          </v-dialog>
        </v-toolbar-items>
      </v-toolbar>
      <v-flex xs12>
        <material-card
          v-if="event"
          color="primary"
        >
          <div slot="header">
            <v-layout
              justify-space-between
              align-center
              row
              wrap
              class="pa-2"
            >
              <div>
                <div class="title font-weight-light mb-2">Asistencia</div>
                <div
                  v-if="showCountdown"
                  class="category"
                >
                  Inicia en:
                  <span v-if="showCountdown && daysTgo * 1 > 0">{{ daysTgo }}:</span>
                  <span v-if="showCountdown && hoursTgo * 1 > 0">{{ hoursTgo }}:</span>
                  <span v-if="showCountdown && ((hoursTgo * 1) + (minsTgo * 1)) > 0">{{ minsTgo }}:</span>
                  <span
                    v-if="showCountdown && (( hoursTgo * 1) + (minsTgo * 1) + (secsTgo * 1)) > 0"
                  >{{ secsTgo }}</span>
                </div>
              </div>

              <v-btn
                :color="activeColor"
                dark
                small
                icon
                @click="event.active = !event.active"
              >
                <v-icon
                  v-if="event.active"
                  color="white"
                >mdi-lock-open-outline</v-icon>
                <v-icon
                  v-else
                  color="white"
                >mdi-lock-clock</v-icon>
              </v-btn>
            </v-layout>
          </div>
          <v-card-text>
            <v-form>
              <v-layout
                row
                wrap
              >
                <v-flex xs12>
                  <v-combobox
                    ref="eventMemberAsist"
                    :disabled="!event.active"
                    v-model="event.memberAsist"
                    :items="members"
                    label="Miembros"
                    small-chips
                    multiple
                    cache-items
                  />
                </v-flex>
                <v-flex xs12>
                  <v-combobox
                    ref="eventGuests"
                    :disabled="!event.active"
                    v-model="computedEventGuests"
                    :items="guests"
                    label="Invitados"
                    small-chips
                    multiple
                    cache-items
                  />
                </v-flex>
              </v-layout>
            </v-form>
            <v-card-actions slot="actions">
              <v-spacer></v-spacer>
              <v-btn
                :small="responsive"
                color="success"
                @click="saveAdminEvent"
              >Guardar</v-btn>
            </v-card-actions>
          </v-card-text>
        </material-card>
      </v-flex>

      <v-flex xs12/>
    </v-layout>
  </v-container>
</template>

<script>
import { mapGetters } from 'vuex'
export default {
  name: 'AdminEvent',
  data() {
    return {
      event: this.$store.getters.getEventById(this.$route.params.id),
      daysTgo: null,
      hoursTgo: null,
      minsTgo: null,
      secsTgo: null,
      showCountdown: false,
      watch: false,
      activeColor: 'warning',
      registerGuestModal: false,
      responsive: false
    }
  },
  computed: {
    computedEventGuests: {
      get: function() {
        const arrayGuestsObjs = this.event.guests.map(id => {
          const guestById = this.$store.getters.getAllGuests.filter(
            guest => guest._id === id
          )[0]

          if (guestById) {
            return guestById
          } else {
            return {}
          }
        })

        if (arrayGuestsObjs) {
          const guestName = arrayGuestsObjs.map(guest => guest.name)

          if (guestName) {
            return guestName
          } else {
            return []
          }
        } else {
          return []
        }
      },
      set: function(payload) {
        const index = this.$store.state.events
          .map(event => {
            return event._id
          })
          .indexOf(this.$route.params.id)

        const guestsObjs = payload
          .map(name => {
            return this.$store.getters.getAllGuests.filter(
              guest => guest.name === name
            )[0]
          })
          .map(guest => guest._id)

        const topayload = {
          index: index,
          guestsIdTosave: guestsObjs
        }

        this.$store.commit('setGuestsOnEvent', topayload)
      }
    },
    guests() {
      return this.$store.getters.getAllGuests.map(guest => guest.name)
    },
    members() {
      return this.$store.getters.getAllMembers.map(member => member.name)
    },
    chekcActive() {
      if (this.event) return this.event.active
    },

    eventState() {
      const eventSt = this.$store.getters.getEventById(this.$route.params.id)

      if (eventSt) {
        const setState = this.setState(eventSt)
        if (setState) {
          this.countDown(eventSt.eventDate, eventSt.eventTime)
        }
      }
      return eventSt
    }
  },
  watch: {
    eventState(val) {
      this.event = val
    },
    chekcActive(val) {
      if (val === true) {
        this.activeColor = 'success'
      } else {
        this.activeColor = 'warning'
      }
    }
  },
  created() {},
  mounted() {
    this.setActive()
  },
  beforeDestroy() {
    clearInterval(this.x)
  },
  methods: {
    onResponsiveInverted() {
      if (window.innerWidth < 991) {
        this.responsive = true
      } else {
        this.responsive = false
      }
    },
    closeModal() {
      this.registerGuestModal = false
    },
    setActive() {
      if (this.eventState) {
        const [ye, mo, da] = this.eventState.eventDate.split('-')
        const [ho, mi] = this.eventState.eventTime.split(':')
        const eventDate = new Date(ye, mo - 1, da, ho, mi, 0, 0)
        const now = Date.now()

        if (eventDate - now <= 0) {
          const payload = {
            id: this.event._id,
            active: true
          }
          this.$store.commit('setActive', payload)
          this.showCountdown = false
        }
      }
    },
    setState(localState) {
      return (this.event = localState)
    },
    countDown(date, time) {
      const vm = this
      const [ye, mo, da] = date.split('-')
      const [ho, mi] = time.split(':')

      return (this.x = setInterval(function() {
        function pad(n) {
          return n < 10 ? '0' + n : n
        }
        const eventDate = new Date(ye, mo - 1, da, ho, mi, 0, 0)
        const now = Date.now()
        // const eventDate = new Date(date)
        const timeTo = eventDate - now
        if (timeTo > 0) {
          vm.showCountdown = true
          vm.daysTgo = Math.floor(timeTo / (1000 * 60 * 60 * 24))
          vm.hoursTgo = pad(
            Math.floor((timeTo % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60))
          )
          vm.minsTgo = pad(
            Math.floor((timeTo % (1000 * 60 * 60)) / (1000 * 60))
          )
          vm.secsTgo = pad(Math.floor((timeTo % (1000 * 60)) / 1000))
        }
        if (timeTo === 0) {
          vm.setActive()
          clearInterval(vm.x)
          vm.showCountdown = false
        }
      }, 1000))
    },

    saveAdminEvent() {}
  },
  mounted() {
    this.onResponsiveInverted()
    window.addEventListener('resize', this.onResponsiveInverted)
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.onResponsiveInverted)
  }
}
</script>

<style lang="scss">
@import '../styles/index.scss';
.admin-event .v-toolbar .v-toolbar__content {
  margin-left: 0;
}
</style>

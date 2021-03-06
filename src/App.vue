<template lang="jade">
#app
  landing(v-if='!user')
  div(v-if="user")
    b-navbar(toggleable="md", type="light", variant="light")
      b-navbar-toggle(target="nav_collapse")
      b-navbar-brand(to="/") KanStack
      b-collapse(is-nav, id="nav_collapse")
        b-navbar-nav
          b-nav-item(to="/boards/new") New Board
        b-navbar-nav.ml-auto
          b-nav-item-dropdown(right, :text="user.username")
            b-dropdown-item(href="javascript:;", @click="signOut") Sign Out
          <img :src="user.avatarUrl() ? user.avatarUrl() : '/avatar-placeholder.png'" class="img-fluid rounded-circle avatar mt-1">
  div(v-if="user && importing")
    .row.mt-3
      .col-12.text-center.mt-3
        h1.mb-3
          font-awesome-icon(:icon="spinnerIcon" pulse)
        h1 Loading ...
  div.h-100(v-if="user && !importing")
    router-view
    //- small.creds
    //-   | Source code on
    //-   a(href='https://github.com/blockstack/blockstack-todos', target='_blank') Github

</template>

<script>
import FontAwesomeIcon from '@fortawesome/vue-fontawesome'
import spinnerIcon from '@fortawesome/fontawesome-free-solid/faSpinner'

const blockstack = require('blockstack')

import Landing from './components/Landing.vue'

import db from './db'

export default {
  name: 'app',
  components: {Landing, FontAwesomeIcon},
  async mounted () {
    if (blockstack.isUserSignedIn()) {
      this.userData = blockstack.loadUserData()
      this.signIn()
    } else if (blockstack.isSignInPending()) {
      blockstack.handlePendingSignIn()
      .then((userData) => {
        this.userData = userData
        this.signIn()
      })
    }
  },
  methods: {
    signOut () {
      blockstack.signUserOut(window.location.href)
      this.user = null
      this.$router.push('/')
    },
    async signIn () {
      this.user = new blockstack.Person(this.userData.profile)
      this.user.username = this.userData.username || this.user._profile.name
      try {
        await db.blockstackImport()
      } catch (error) {
        // possibly offline
        this.importing = false
      }
      this.importing = false
    }
  },
  data () {
    return {
      user: null,
      blockstack: blockstack,
      importing: true,
      spinnerIcon,
      userData: null
    }
  }
}
</script>

<style lang="sass" scoped>
.avatar
  max-height: 30px
</style>

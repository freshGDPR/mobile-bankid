<template>
  <div >
    <figure class="item">
      <div class="thumbnail" @click="handleMobileBankId">
        <img src="./bankid_256x256.png" height="100" alt="mobile_bankid" />
      </div>
      <figcaption class="caption" id="mobilecaption">Mobile BankID</figcaption>
    </figure>

    <b-modal ref="modal1"
             title="Mobile BankID"
             header-bg-variant="success"
             header-text-variant="light"
             ok-only
             hide-footer
             >
      <!-- TODO Recommended User Messages -->
      <b-card no-body>
        <b-tabs card no-key-nav v-model="tabIndex">
          <b-tab no-body title="1" active>
            <b-form inline>
              <label class="mr-sm-2" for="personalNumber">Personal number:</label>
              <b-input class="mb-2 mr-sm-2 mb-sm-0" id="personalNumber" v-model="personalNumber" />
            </b-form>
          </b-tab>
          <b-tab no-body title="2">
            <b-progress :value="counter" :max="max" animated></b-progress>
            <h3 id="bar">{{ message }}</h3>
          </b-tab>
        </b-tabs>
      </b-card>

      <!-- Control buttons-->
      <div class="text-center">
        <b-button-group class="mt-2">
          <!-- <b-btn @click="tabIndex--">Previous</b-btn> -->
          <b-btn @click="handleNext">Next</b-btn>
        </b-button-group>
        <br>
        <span class="text-muted">Current Tab: {{tabIndex}}</span>
      </div>

    </b-modal>

    <b-modal ref="modal2"
             title="Mobile BankID"
             header-bg-variant="danger"
             header-text-variant="light"
             ok-only
             >
      <h3>No such request</h3>
    </b-modal>

  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'mobile-bankid',
  props: [
    'hash',
    'personalNumber'
  ],
  data () {
    return {
      counter: 45,
      max: 100,
      tabIndex: 1,
      message: 'Start your BankID app',
      timer: '',
      token: 'no token (jet)'
    }
  },
  methods: {
    handleMobileBankId () {
      /* eslint-disable no-console */
      console.log('Mobile BankID')
      /* eslint-disable no-console */
      console.log('hash: ')
      /* eslint-disable no-console */
      console.log(this.hash)
      /* eslint-disable no-console */
      console.log('personalNumber: ')
      /* eslint-disable no-console */
      console.log(this.personalNumber)

      axios({
        method: 'get',
        url: ((process.env.NODE_ENV === 'production') ? 'https://api.freshgdpr.com' : 'http://localhost:9090') + '/mobile-bankid',
        params: (this.hash) ? {
          h: this.hash
        } : {
          personal_number: this.personalNumber
        }
      })
        .then(response => {
          /* eslint-disable no-console */
          console.log('mobile-bankid ' + JSON.stringify(response))

          this.tabIndex = 1
          this.token = response['data']['token']
          this.$refs.modal1.show()

          this.timer = setInterval(this.fetchMessage, 500) // 0.5 sec
        })
      /* eslint-disable no-unused-vars */
        .catch(response => {
          this.$refs.modal2.show()
        })
    },
    fetchMessage () {
      /* eslint-disable no-console */
      console.log('fetchMessage ' + this.token)

      axios({
        method: 'get',
        url: ((process.env.NODE_ENV === 'production') ? 'https://api.freshgdpr.com' : 'http://localhost:9090') + '/collect',
        params: {
          t: this.token
        }
      })
        .then(response => {
          /* eslint-disable no-console */
          console.log('mobile-bankid collect ' + JSON.stringify(response))
          var state = response['data']['state']
          // SENT_TO_BANKID = 0
          // HAS_MESSAGE    = 1
          // ACCESS_GRANTED = 2
          // DENIED         = 3
          if (state === 1) {
            this.message = response['data']['message']
          } else if (state === 2) {
            // TODO as in login
            clearInterval(this.timer)
            this.$refs.modal1.hide()
            this.$emit('access-granted', response)
          } else if (state === 3) {
            /* eslint-disable no-console */            
            console.log('mobile-bankid login denied')
          }
        })
    },
    handleNext () {
      switch (this.tabIndex) {
        case 0:
          /* do nothing */
          break
        case 1:
          /* eslint-disable no-console */
          console.log('Next tabIndex 1')
          break
      }
    }
  }
}
</script>

<style>
/* XXX */
/* reset this */
.card-footer {
  all: initial;
  * {
    all: unset;
  }
}
/* hide */
.card-footer {
  display: none;
  visibility: hidden;
}
/* reset this */
.card-header {
  all: initial;
  * {
    all: unset;
  }
}
/* hide */
.card-header {
  display: none;
  visibility: hidden;
}
/* reset this */
.card-header-tabs {
  all: initial;
  * {
    all: unset;
  }
}
/* hide */
.card-header-tabs {
  display: none;
  visibility: hidden;
}
#bar {
  all: initial;
  * {
    all: unset;
  }
}
#bar {
  font: 20px; /* Invalid property value */ 
}
.thumbnail img {
  cursor:pointer;
}
</style>

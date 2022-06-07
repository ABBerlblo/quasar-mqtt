<template>
  <q-page class="bg-dark">
    <q-tabs no-caps active-color="info" indicator-color="transparent"
      class="text-primary bg-primary q-pt-md q-pb-xs q-px-md" v-model="tab" align="justify">
      <q-tab name="Controller" label="Joystick" class="underline text" id="Controller" />
      <q-tab name="Log" label="Log" class="underline" id="Log" />
    </q-tabs>

    <q-tab-panels v-model="tab" animated class="bg-dark" @mouseup="JoyEnd" @touchend="JoyEnd">

      <q-tab-panel name="Controller" style="height:92vh">

        <div class="absolute bg-primary fixed-center circle" style="height:360px; width:360px;" @mousedown="JoyBegin"
          @touchstart="JoyBegin">
        </div>

        <div ref="PointOfFocus" class="absolute bg-info fixed-center circle" style="height:120px;width:120px;"
          @mousedown="JoyBegin" @touchstart="JoyBegin">
        </div>

      </q-tab-panel>

      <q-tab-panel name="Log" style="height:92vh; display: flex; align-items:center; justify-content:center;">
        <div style="width:60vw; height:60vh; padding-left: 2vw; padding-bottom: 2vh; padding-top: 2vh;"
          class="text-info bg-grey-9" id="messages">
          <span v-for="command in logrecord" :key="command" class="text-h6">{{ command }} <br></span>
        </div>
      </q-tab-panel>
    </q-tab-panels>
  </q-page>
</template>

<script>
import { client } from "../boot/mqtt-boot"

export default {
  created () {
    client.on("connect", () => {
      console.log("Conntected!")
    })
  },
  mounted () {
    this.orgin_y = this.$refs.PointOfFocus.getBoundingClientRect().bottom - 120
    this.orgin_x = this.$refs.PointOfFocus.getBoundingClientRect().left + 60

    if (process.browser) {
      window.addEventListener('keydown', (e) => {
        console.log(e.key)
        if (e.key === ' ') {
          this.handbr = !this.handbr
        }
        if (e.key === 'Shift' && !this.handbr) {
          this.fastest = true
        }
        if (e.key === 'z' || e.key === 'Z') {
          this.bad = !this.bad
          console.log('Strife: ' + this.bad)
        }
        if (e.key === 'ArrowUp' && !this.handbr || e.key === 'w' && !this.handbr) {
          if (this.force == 0 && this.fastest) {
            this.force = 1080
          }
          if (this.force == 0 && !this.fastest) {
            this.force = 180
          }
        }
        if (e.key === 'ArrowDown' && !this.handbr || e.key === 's' && !this.handbr) {
          if (this.force == 0 && this.fastest) {
            this.force = -1080
          }
          if (this.force == 0 && !this.fastest) {
            this.force = -180
          }
        }
        if (e.key === 'ArrowRight' || e.key === 'd') {
          this.turnangle = 180
        }
        if (e.key === 'ArrowLeft' || e.key === 'a') {
          this.turnangle = 0
        }
      })
    }
    if (process.browser) {
      window.addEventListener('keyup', (e) => {

        if (e.key === 'Shift' && !this.handbr) {
          this.fastest = false
        }
        if (e.key === 'ArrowUp' || e.key === 'w' || e.key === 'ArrowDown' || e.key === 's') {
          this.force = 0
        }
        if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'ArrowLeft' || e.key === 'a') {
          this.turnangle = 90
        }
      })
    }

    if (process.browser) {
      const center_y = this.$refs.PointOfFocus.getBoundingClientRect().bottom - 60
      const center_x = this.$refs.PointOfFocus.getBoundingClientRect().left + 60
      window.addEventListener('mousemove', (e) => {
        const x_offset = e.clientX - center_x
        const y_offset = e.clientY - center_y
        const h = Math.sqrt(x_offset * x_offset + y_offset * y_offset)

        let new_x_offset, new_y_offset

        if (h <= 180) {
          new_x_offset = x_offset
          new_y_offset = y_offset
        } else {
          new_x_offset = x_offset * 180 / h
          new_y_offset = y_offset * 180 / h
        }

        this.PointOfFocusChangeY = `${Math.round(new_y_offset + center_y - 60)}`
        this.PointOfFocusChangeX = `${Math.round(new_x_offset + center_x)}`
      })
    }

    if (process.browser) {
      const center_y = this.$refs.PointOfFocus.getBoundingClientRect().bottom - 60
      const center_x = this.$refs.PointOfFocus.getBoundingClientRect().left + 60
      window.addEventListener('touchmove', (e) => {
        const x_offset = e.clientX - center_x
        const y_offset = e.clientY - center_y
        const h = Math.sqrt(x_offset * x_offset + y_offset * y_offset)

        let new_x_offset, new_y_offset

        if (h <= 180) {
          new_x_offset = x_offset
          new_y_offset = y_offset
        } else {
          new_x_offset = x_offset * 180 / h
          new_y_offset = y_offset * 180 / h
        }

        this.PointOfFocusChangeY = `${Math.round(new_y_offset + center_y - 60)}`
        this.PointOfFocusChangeX = `${Math.round(new_x_offset + center_x)}`
      })
    }
  },

  data () {
    return {
      JoyInUse: false,
      orgin_y: 0,
      orgin_x: 0,
      PointOfFocusChangeY: 0,
      PointOfFocusChangeX: 0,

      JoyTurn: 90,

      JoyDrive: 0,
      DriveInterval: 0,

      receivedMessage: "",
      publishMessage: "",
      chatMessages: [],
      tab: "Controller",
      logrecord: [],

      force: 0,
      fastest: false,
      bad: false,
      handbr: false,
      turnangle: 90,
    }
  },

  methods: {
    publish (action, amplitude) {
      client.publish("erlblo/" + action, amplitude.toString())
    },
    JoyBegin () {
      this.JoyInUse = true
      console.log(this.JoyInUse)
    },
    JoyEnd () {
      this.JoyInUse = false
      console.log(this.JoyInUse)

      this.$refs.PointOfFocus.style.left = `${this.orgin_x}px`
      this.$refs.PointOfFocus.style.top = `${this.orgin_y - 8}px`
      this.publish("drive", 0)
      this.logrecord.push('Speed: ' + 0)
      this.publish("turn", 90)
      this.logrecord.push('Turned: ' + 90 + '°')
    },
  },
  watch: {
    PointOfFocusChangeY (PointOfFocusChangeY) {
      const PointOfFocus = this.$refs.PointOfFocus
      if (this.JoyInUse) {
        PointOfFocus.style.top = `${PointOfFocusChangeY}px`
        this.DriveInterval = Math.round(((PointOfFocusChangeY - this.orgin_y) * -1 * (1080 / 180)))
      }
    },
    DriveInterval (DriveInterval) {
      if (-180 < DriveInterval && DriveInterval < 180) {
        this.JoyDrive = 0
      }
      else if (-360 < DriveInterval && DriveInterval < -180) {
        this.JoyDrive = -360
      }
      else if (-720 < DriveInterval && DriveInterval < -360) {
        this.JoyDrive = -720
      }
      else if (-1080 < DriveInterval && DriveInterval < -720) {
        this.JoyDrive = -1080
      }
      else if (180 < DriveInterval && DriveInterval < 360) {
        this.JoyDrive = 360
      }
      else if (360 < DriveInterval && DriveInterval < 720) {
        this.JoyDrive = 720
      }
      else if (720 < DriveInterval && DriveInterval < 1080) {
        this.JoyDrive = 1080
      }
    },
    JoyDrive (JoyDrive) {
      console.log(this.JoyDrive)
      this.publish("drive", JoyDrive)
      this.logrecord.push('Speed: ' + JoyDrive)
    },
    PointOfFocusChangeX (PointOfFocusChangeX) {
      const PointOfFocus = this.$refs.PointOfFocus
      if (this.JoyInUse) {
        PointOfFocus.style.left = `${PointOfFocusChangeX}px`
        this.JoyTurn = Math.round((PointOfFocusChangeX - this.orgin_x) / 2) + 90
      }
    },
    JoyTurn (JoyTurn) {
      console.log('Turn: ' + this.JoyTurn)
      this.publish("turn", JoyTurn)
      this.logrecord.push('Turned: ' + JoyTurn + '°')

    },
    force (force) {
      this.publish("drive", force)
      this.logrecord.push('Speed: ' + force)
      console.log('Speed: ' + this.force)
    },
    fastest: function (fastest) {
      if (fastest) {
        this.force = this.force * 6
        this.logrecord.push('Gas: ' + this.fastest)
        console.log('Gas: ' + this.fastest)
      }
      if (!fastest) {
        this.force = this.force / 6
        this.logrecord.push('Gas: ' + this.fastest)
        console.log('Gas: ' + this.fastest)
      }
    },
    handbr: function (handbr) {
      if (handbr) {
        this.force = 0
        this.logrecord.push('Break: ' + this.handbr)
        console.log('Break: ' + this.handbr)
      }
      if (!handbr) {
        this.force = 0
        this.logrecord.push('Break: ' + this.handbr)
        console.log('Break: ' + this.handbr)
      }
    },
    turnangle (turnangle) {
      if (this.bad) {
        this.publish("strife", turnangle)
        this.logrecord.push('Strifing: ' + turnangle + '°')
        console.log('Strifing: ' + turnangle + '°')
      }
      if (!this.bad) {
        this.publish("turn", turnangle)
        this.logrecord.push('Turned: ' + turnangle + '°')
        console.log('Turned: ' + turnangle + '°')
      }
    },
  },
  destroyed () {
    if (process.browser) {
      window.removeEventListener('keydown')
    }
    if (process.browser) {
      window.removeEventListener('keyup')
    }
    if (process.browser) {
      window.removeEventListener('mousemove')
    }
    if (process.browser) {
      window.removeEventListener('touchmove')
    }
  },
}
</script>

<style scoped>
.circle {
  border-radius: 50%;
  border: solid 4px rgb(49, 204, 236);
}

.underline {
  position: relative;
  color: #000000;
  text-decoration: none;
}

.underline:focus {
  color: #000000;
}

.underline::before {
  content: "";
  position: absolute;
  display: block;
  width: 80%;
  height: 2.5px;
  bottom: 5px;
  left: 10%;
  background-color: #0aaab0;
  transform: scaleX(0);
  transition: transform 0.3s ease;
}

.underline:hover::before {
  transform: scaleX(1);
}

#messages {
  display: inline-block;
  overflow-y: scroll;
  border-radius: 4px;
}

#messages span {
  overflow-y: scroll;
  /* color: #08aeb4; */
}
</style>

<template>
  <div>
    <button
      @click.prevent="hang()"
      class="SocialSharing__link call-hang"
      :class="inCall ? 'call-hang-active' : ''"
      v-if="inCall"
    >
      <i class="fas fa-phone"></i>
    </button>
    <button
      @click.prevent="dial(number)"
      class="SocialSharing__link call-dial is-animating"
      v-else
    >
      <i class="fas fa-phone"></i>
    </button>
    <audio
      src="@/assets/sounds/ringing.mp3"
      ref="ringing"
      style="display: none !important;"
    ></audio>
  </div>
</template>

<script>
const PH_ST_CONNECTED = "connected";
const PH_ST_DISCONNECTED = "disconnected";
const PH_ST_REGISTRATIONFAILED = "registrationFailed";
const PH_ST_UNREGISTERED = "unregistered";
const PH_ST_REGISTERED = "registered";

const SE_ST_PROGRESS = "progress";
const SE_ST_ACCEPTED = "accepted";
const SE_ST_CANCELED = "canceled";
const SE_ST_TERMINATED = "terminated";
const SE_ST_FAILED = "failed";
const SE_ST_REJECTED = "rejected";
const SE_ST_BYE = "bye";
const SE_ST_DTMF = "dtmf";

export default {
  props: [
    "sipServer",
    "authorizationUser",
    "password",
    "displayName",
    "userAgentString",
    "stunServers",
    "logLevel",
    "avatar",
    "number"
  ],

  data() {
    return {
      phone: null,
      session: null,
      inCall: false,
      state: "Iniciando",
      callState: "",
      original_avatar: ""
    };
  },

  mounted() {
    this.original_avatar = this.avatar;
    this.$refs["ringing"].loop = true;

    this.phone = this.createPhone({
      sipServer: this.sipServer,
      authorizationUser: this.authorizationUser,
      password: this.password,
      displayName: this.displayName,
      userAgentString: this.userAgentString,
      stunServers: this.stunServers,
      logLevel: this.logLevel
    });

    this.phone.on(PH_ST_CONNECTED, e => {
      this.log(PH_ST_CONNECTED, e);
      this.state = PH_ST_CONNECTED;
    });

    this.phone.on(PH_ST_DISCONNECTED, e => {
      this.log(PH_ST_CONNECTED, e);
      this.state = PH_ST_DISCONNECTED;
    });

    this.phone.on(PH_ST_REGISTRATIONFAILED, e => {
      this.log(PH_ST_CONNECTED, e);
      this.state = PH_ST_REGISTRATIONFAILED;
    });

    this.phone.on(PH_ST_UNREGISTERED, e => {
      this.log(PH_ST_CONNECTED, e);
      this.state = PH_ST_UNREGISTERED;
    });

    this.phone.on(PH_ST_REGISTERED, e => {
      this.log(PH_ST_CONNECTED, e);
      this.state = PH_ST_REGISTERED;
    });
  },

  beforeDestroy() {
    this.phone.unregister();
  },

  methods: {
    createPhone(conf) {
    /* eslint-disable */
      return new SIP.UA({
        password: conf.password,
        authorizationUser: conf.authorizationUser,
        displayName: conf.displayName,
        userAgentString: conf.userAgentString,
        uri:
          "sip:" +
          conf.authorizationUser +
          "@" +
          this.extractDomain(conf.sipServer),
        transportOptions: {
          wsServers: [conf.sipServer],
          traceSip: false,
          maxReconnectionAttempts: 30,
          reconnectionTimeout: 5
        },
        hackWssInTransport: true,
        registerExpires: 30,
        hackIpInContact: true,
        log: {
          level: conf.logLevel
        },
        stunServers: conf.stunServers,
        sessionDescriptionHandlerFactoryOptions: {
          constraints: {
            audio: true,
            video: false
          }
        }
      });
    },
    /* eslint-enable */
    dial(dialCall) {
      this.log("Dial", dialCall);

      this.session = this.phone.invite(dialCall);
      this.inCall = true;
      this.callState = "Calling";
      this.$refs["ringing"].play();

      this.session.on(SE_ST_PROGRESS, e => {
        this.log(SE_ST_PROGRESS, e);
      });

      this.session.on(SE_ST_ACCEPTED, e => {
        this.log(SE_ST_ACCEPTED, e);
        this.plugAudio(this.session.sessionDescriptionHandler.peerConnection);
        this.callState = SE_ST_ACCEPTED;
        this.$refs["ringing"].pause();
      });

      this.session.on(SE_ST_CANCELED, e => {
        this.log(SE_ST_CANCELED, e);
        this.inCall = false;
        this.callState = SE_ST_CANCELED;
        this.$refs["ringing"].pause();
        this.setOriginalAvatar();
      });

      this.session.on(SE_ST_TERMINATED, () => {
        this.log(SE_ST_TERMINATED);
        this.inCall = false;
        this.callState = SE_ST_TERMINATED;
        this.$refs["ringing"].pause();
        setTimeout(() => {
          this.callState = "";
        }, 3000);
      });

      this.session.on(SE_ST_FAILED, e => {
        this.log(SE_ST_FAILED, e);
        this.inCall = false;
        this.callState = SE_ST_FAILED;
        this.setOriginalAvatar();
      });

      this.session.on(SE_ST_REJECTED, e => {
        this.log(SE_ST_REJECTED, e);
        this.inCall = false;
        this.callState = SE_ST_REJECTED;
        this.setOriginalAvatar();
      });

      this.session.on(SE_ST_BYE, e => {
        this.log(SE_ST_BYE, e);
      });

      this.session.on(SE_ST_DTMF, e => {
        this.log(SE_ST_DTMF, e);
      });
    },

    hang() {
      this.session.terminate();
    },

    extractDomain(url) {
      let domain;
      if (url.indexOf("://") > -1) {
        domain = url.split("/")[2];
      } else {
        domain = url.split("/")[0];
      }
      return domain.split(":")[0];
    },

    plugAudio(pc) {
      const voice = new Audio();
      if (pc.getReceivers) {
        const stream = new window.MediaStream();
        pc.getReceivers().forEach(function(receiver) {
          if (receiver.track) {
            stream.addTrack(receiver.track);
          }
        });
        voice.srcObject = stream;
      } else {
        voice.srcObject = pc.getRemoteStreams()[0];
      }
      voice.play();
    },

    log(...args) {
      if (this.logLevel > 0) console.log(args);
    }
  }
};
</script>

<style lang="scss" scoped>
.SocialSharing__link {
  border: none;
  padding: 0;

  &:focus {
    outline: 0;
    box-shadow: none;
  }
}

.call-hang i {
  color: #eb7244;
  position: relative;
}

.call-dial i {
  color: #0097a7;
}

.call-dial.is-animating {
  animation: phone-outer 3000ms infinite;

  &::after {
    animation: phone-icon 3000ms infinite;
  }
}

.call-dial.call-dial.is-animating i {
  animation: ringing 2000ms infinite;
}

@keyframes rotate-phone {
  0% {
    transform: translate(0deg, 0deg);
  }
  50% {
    transform: translate(45deg, 0deg);
  }
  100% {
    transform: translate(0deg, 0deg);
  }
}

@keyframes phone-outer {
  0% {
    transform: translate3d(0, 0, 0) scale(1);
  }
  33.3333% {
    transform: translate3d(0, 0, 0) scale(1.3);
  }
  66.6666% {
    transform: translate3d(0, 0, 0) scale(1);
  }
  100% {
    transform: translate3d(0, 0, 0) scale(1);
  }
}

@keyframes phone-inner {
  0% {
    opacity: 1;
    transform: translate3d(0, 0, 0) scale(0);
  }
  33.3333% {
    opacity: 1;
    transform: translate3d(0, 0, 0) scale(0.9);
  }
  66.6666% {
    opacity: 0;
    transform: translate3d(0, 0, 0) scale(0);
  }
  100% {
    opacity: 0;
    transform: translate3d(0, 0, 0) scale(0);
  }
}

@keyframes phone-icon {
  0% {
    transform: translate3d(0em, 0, 0);
  }
  2% {
    transform: translate3d(0.01em, 0, 0);
  }
  4% {
    transform: translate3d(-0.01em, 0, 0);
  }
  6% {
    transform: translate3d(0.01em, 0, 0);
  }
  8% {
    transform: translate3d(-0.01em, 0, 0);
  }
  10% {
    transform: translate3d(0.01em, 0, 0);
  }
  12% {
    transform: translate3d(-0.01em, 0, 0);
  }
  14% {
    transform: translate3d(0.01em, 0, 0);
  }
  16% {
    transform: translate3d(-0.01em, 0, 0);
  }
  18% {
    transform: translate3d(0.01em, 0, 0);
  }
  20% {
    transform: translate3d(-0.01em, 0, 0);
  }
  22% {
    transform: translate3d(0.01em, 0, 0);
  }
  24% {
    transform: translate3d(-0.01em, 0, 0);
  }
  26% {
    transform: translate3d(0.01em, 0, 0);
  }
  28% {
    transform: translate3d(-0.01em, 0, 0);
  }
  30% {
    transform: translate3d(0.01em, 0, 0);
  }
  32% {
    transform: translate3d(-0.01em, 0, 0);
  }
  34% {
    transform: translate3d(0.01em, 0, 0);
  }
  36% {
    transform: translate3d(-0.01em, 0, 0);
  }
  38% {
    transform: translate3d(0.01em, 0, 0);
  }
  40% {
    transform: translate3d(-0.01em, 0, 0);
  }
  42% {
    transform: translate3d(0.01em, 0, 0);
  }
  44% {
    transform: translate3d(-0.01em, 0, 0);
  }
  46% {
    transform: translate3d(0em, 0, 0);
  }
}

@keyframes ringing {
  0% {
  }
  10% {
    -webkit-transform: rotate(5deg);
    -moz-transform: rotate(5deg);
    -o-transform: rotate(5deg);
  }
  20% {
    -webkit-transform: rotate(-5deg);
    -moz-transform: rotate(-5deg);
    -o-transform: rotate(-5deg);
  }
  30% {
    -webkit-transform: rotate(5deg);
    -moz-transform: rotate(5deg);
    -o-transform: rotate(5deg);
  }
  40% {
    -webkit-transform: rotate(-5deg);
    -moz-transform: rotate(-5deg);
    -o-transform: rotate(-5deg);
  }
  50% {
    -webkit-transform: rotate(5deg);
    -moz-transform: rotate(5deg);
    -o-transform: rotate(5deg);
  }
  60% {
    -webkit-transform: rotate(-5deg);
    -moz-transform: rotate(-5deg);
    -o-transform: rotate(-5deg);
  }
  70% {
    -webkit-transform: rotate(5deg);
    -moz-transform: rotate(5deg);
    -o-transform: rotate(5deg);
  }
  80% {
    -webkit-transform: rotate(-5deg);
    -moz-transform: rotate(-5deg);
    -o-transform: rotate(-5deg);
  }
  90% {
    -webkit-transform: rotate(5deg);
    -moz-transform: rotate(5deg);
    -o-transform: rotate(5deg);
  }
  100% {
    -webkit-transform: rotate(-5deg);
    -moz-transform: rotate(-5deg);
    -o-transform: rotate(-5deg);
  }
}
</style>

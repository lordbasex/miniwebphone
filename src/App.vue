<template>
  <div id="app">
    <webphone
      :sipServer="config.sipServer"
      :authorizationUser="config.authorizationUser"
      :password="config.password"
      :displayName="config.displayName"
      :userAgentString="config.userAgentString"
      :stunServers="config.stunServers"
      :logLevel="config.logLevel"
      :avatar="config.avatar"
      :number="config.number"
    />
  </div>
</template>

<script>
import Webphone from "./components/Webphone.vue";

export default {
  name: "App",
  components: {
    Webphone
  },
  data: () => ({
    config: {
      sipServer: process.env.VUE_APP_SIP_SERVER,
      authorizationUser: process.env.VUE_APP_AUTHORIZATION_USER,
      password: process.env.VUE_APP_AUTHORIZATION_PASSWORD,
      displayName: process.env.VUE_APP_DISPLAY_NAME,
      userAgentString: process.env.VUE_APP_USER_AGENT,
      stunServers: process.env.VUE_APP_STUN_SERVERS,
      logLevel: process.env.VUE_APP_LOG_LEVEL,
      avatar: process.env.VUE_APP_AVATAR,
      number: process.env.VUE_APP_NUMBER
    }
  })
};
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
}

.SocialSharing__link {
  background: transparent;
  font-size: 22px;
  line-height: 22px;
  font-weight: 300;

  &:hover {
    outline: 0;
    text-decoration: none;
  }
}

.SocialSharing__link i {
  background: #fff;
  border-radius: 4px;
  color: #0097a7;
  width: 32px;
  height: 32px;
  font-size: 20px;
  display: block;
  line-height: 32px;
  text-align: center;
  margin: 6px 3px;
  transition: all 0.3s;
  position: relative;
  right: 3px;
  box-shadow: -1px -1px 4px rgba(0, 0, 0, 0.3);

  &:hover {
    right: 6px;
  }
}
</style>

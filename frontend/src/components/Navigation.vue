
<template>
  <div>
    <nav class="navbar navbar-default">
      <div class="container-fluid">
        <div class="navbar-header">
          <div class="img">
            <router-link to="/">
              <img class="navbar-brand" src="../assets/logo1.png"
            /></router-link>
          </div>
          <button
            type="button"
            class="navbar-toggle collapsed"
            data-toggle="collapse"
            data-target="#bs-example-navbar-collapse-1"
            aria-expanded="false"
          >
            <span class="sr-only">Toggle navigation</span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
          </button>
        </div>
        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
          <ul class="nav navbar-nav navbar-right">
            <!-- Bouton Seconnecter --------->
            <li
              v-if="$store.state.isConnected == false"
              my_title=" Se connecter"
              class="infobulle"
            >
              <router-link to="/"><i class="fas fa-plug"></i></router-link>
            </li>

            <!-- Bouton Forum --------------->
            <li my_title=" Forum" class="infobulle">
              <router-link to="/Posts">
                <i class="fas fa-people-arrows"></i>
                <!--
                <img src="../assets/icon.png" alt="" width="30" />
              -->
              </router-link>
            </li>

            <!-- Bouton Utilisateurs ---------->
            <li my_title=" Utilisateurs" class="infobulle">
              <router-link to="/Users">
                <i class="fas fa-users"></i
              ></router-link>
            </li>

            <!-- Bouton Mon profil ---------------->
            <li
              v-if="$store.state.isConnected == true"
              my_title=" Mon profil"
              class="infobulle myProfil"
            >
              <router-link :to="{ name: 'Profil', params: { id: userId } }">
                <p
                  v-if="$store.state.isConnected == true"
                  class="connected_circle"
                >
                  <i class="fas fa-circle"></i>
                </p>
                <img
                  v-if="$store.state.isConnected == true"
                  :src="
                    require(`./../../../backend/images/${$store.state.userConnected.profil_picture}`)
                  "
                  width="30"
                  height="30"
                  alt=""
                />
                <i v-else class="fas fa-user-tie"></i>
              </router-link>
            </li>

            <!-- Bouton Se déconnecter ------------------>
            <li
              v-if="$store.state.isConnected == true"
              @click="deconnected"
              my_title=" Se déconnecter"
              class="infobulle"
            >
              <router-link to="/"
                ><i class="fas fa-sign-out-alt"></i
              ></router-link>
            </li>
          </ul>
        </div>
      </div>
    </nav>
  </div>
</template>

<script>
export default {
  name: "Navigation",

  data() {
    return {
      Token: "",
      profil_picture: "",
    };
  },
  computed: {
    userId: function () {
      return this.$store.state.userConnected.userId;
    },
  },
  created() {
    if (JSON.parse(localStorage.getItem("Token"))) {
      this.Token = JSON.parse(localStorage.getItem("Token"));
      this.$store.commit("USER_CONNECTED");
      this.userId = this.Token.userId;
    } else {
      this.deconnected();
    }
  },

  methods: {
    deconnected() {
      localStorage.removeItem("Token");
      this.$store.commit("DECONNEXION");
    },
  },
};
</script>

<style scoped>
body {
  background: #ffd7d7;
  text-align: right;
}

li {
  margin-right: 15px;
}
li i {
  color: #ffd7d7;
  text-shadow: 2px 2px 10px #162948;
}
.isDeconnected {
  color: red;
}
.infobulle img {
  border-radius: 50%;
  box-shadow: 1px 1px 10px black;
  margin-right: 8px;
}
/* --------- Les infos-bulles ------------ */
[my_title] {
  position: relative;
}
[my_title]:hover:after {
  opacity: 1;
  transition: all 0.1s ease 0.5s;
  visibility: visible;
}
[my_title]:after {
  content: attr(my_title);
  position: absolute;
  bottom: 10px;
  right: 60px;
  visibility: hidden;
  font-family: cursive;
  font-weight: 600;
}
[my_title]:after:active {
  visibility: hidden;
}
.connected_circle i {
  color: greenyellow;
  font-size: 12px;
  margin-right: 20px;
}

@media screen and (min-width: 768px) {
  li {
    width: 105px;
    font-size: 0.9em;
  }
  [my_title]:after {
    right: 10px;
    bottom: -4px;
  }
}
</style>

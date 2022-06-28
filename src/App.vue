<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/about">About</router-link>
      <div>
        <span>Version: </span> <span>{{ version }}</span>
      </div>
    </div>
    <router-view />

    <div>
      <span>Is LoggedIn {{ isLoggedIn }}</span>
      <div>
        <h1>=================================</h1>
        <div>
          <span>This is image</span>
          <img :src="profileImg" width="500" height="500" />
        </div>
        <button :disabled="isLoggedIn" v-on:click="logIn">Log In</button>
        <button :disabled="!isLoggedIn" v-on:click="logOut">Log Out</button>
        <div v-for="message in messages" :key="message">
          {{ message }}
        </div>
      </div>

      <div class="wrapper">
        ID Token <button v-on:click="toggleIdTokenBox">Open/Close</button>
        <div v-if="isOpenIdTokenBox">{{ idToken }}</div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import liff from "@line/liff";
import { Vue } from "vue-property-decorator";
import Component from "vue-class-component";
import pjson from "../package.json";

@Component
export default class App extends Vue {
  public isLoggedIn = false;
  public messages = ["start"];
  public version = pjson.version;
  public profileImg = "";
  public isOpenIdTokenBox = false;
  public idToken = "";

  async created(): Promise<void> {
    this.messages.push("create");

    // Error -> `Error: LiffId is not found.` will found when you not login
    await liff.init({ liffId: "1656790536-lEOey5Ox" });

    this.messages.push("liff.isInClient(): " + liff.isInClient());
    this.getUserProfile();

    this.updateLoginStatus();
    this.messages.push("created");

    this.getIdToken();
  }

  logOut(): void {
    this.messages.push("logout");
    liff.logout();
    window.location.reload();
  }

  logIn(): void {
    this.messages.push("login");
    liff.login({ redirectUri: window.location.href });
  }

  updateLoginStatus(): void {
    this.isLoggedIn = liff.isLoggedIn();
  }

  async getUserProfile(): Promise<void> {
    const profile = await liff.getProfile();

    this.messages.push("get profile");
    this.messages.push("profile.userId " + profile.userId);
    this.messages.push("profile.pictureUrl " + profile.pictureUrl);
    this.messages.push("profile.statusMessage " + profile.statusMessage);
    this.messages.push("profile.displayName " + profile.displayName);
    this.profileImg = profile.pictureUrl || "";
  }

  getIdToken(): void {
    this.idToken = liff.getIDToken() ?? "";
  }

  toggleIdTokenBox(): void {
    this.isOpenIdTokenBox = !this.isOpenIdTokenBox;
  }
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#nav {
  padding: 30px;

  a {
    font-weight: bold;
    color: #2c3e50;

    &.router-link-exact-active {
      color: #42b983;
    }
  }
}
</style>

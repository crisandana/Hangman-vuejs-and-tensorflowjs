<template>
  <div id="app">
    <HangmanSVG />
    <UserMode  />
    <!-- <ComputerMode v-else /> -->
    <ErrorsDisplay />
    <!-- <BaseBtn class="switch-mode-btn" @click="switchMode">
      Switch to {{ userMode ? "Computer mode" : "User mode" }}
    </BaseBtn> -->
  </div>
</template>

<script>
import { errors } from "@/shared/errors-observable";
// Components
import ErrorsDisplay from "@/shared/components/ErrorsDisplay";
import HangmanSVG from "./components/HangmanSVG";
import UserMode from "./components/user-mode/UserMode";

//import ComputerMode from "./components/computer-mode/ComputerMode";
//import BaseBtn from "@/shared/components/BaseBtn";


export default {
  name: "app",

  components: {
    HangmanSVG,
    UserMode,
    ErrorsDisplay
    // ComputerMode,
    // BaseBtn
  },

  data: () => ({
    userMode: true
  }),
 
  methods: {
    switchMode() {
      errors.count = 0; // reset errors
      this.userMode = !this.userMode;
    }
  }
   
};
</script>

<style lang="scss">
@import "~@/sass/variables";

html,
body,
#app {
  width: 100%;
  height: 100%;
}

body {
  margin: 0;
  background: $hm_navy;
  color: $hm_yellow;
}

* {
  box-sizing: border-box;
}

kbd {
  border: 1px solid lighten($hm_navy, 90);
  border-radius: 4px;
  display: inline-block;
  padding: 5px 9px;
}
</style>

<style lang="scss" scoped>
#app {
  height: 100%;
  display: grid;
  justify-items: center;
  align-items: start;
  grid-template-rows: 2fr 1fr;
  gap: 10px;
  padding-bottom: 90px;

  ::v-deep .switch-mode-btn {
    position: fixed;
    bottom: 20px;
    left: calc(50% - 123px);
  }
}

@media only screen and (max-width: 600px) {
  ::v-deep .switch-mode-btn {
    right: 20px;
    left: unset !important;
  }
}
</style>

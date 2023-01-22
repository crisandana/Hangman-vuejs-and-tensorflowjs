<template>
  <div class="user-mode">
    <Dialog 
      :show="showDialog" 
      :cancel="cancel" 
      title="How to play?" 
      description="To play this game your mic should be perfect.These are the steps to note before playing this game." 
      />
      <!--The part with the word which should be guessed-->
    <div
      class="word-wrapper"      
      :class="{ success: isWordSolved, fail: maxErrorsExceeded }"
    >
      <DisplayWordChars :chars="chars" :chars-clicked="charsClicked" />

      <button v-show="isWordSolved || maxErrorsExceeded" @click="startGame">
        New Game
      </button>
      <!-- How To Play, Button with informations-->
      <BaseBtn class="switch-mode-btn" @click="open()">
        How to play?
      </BaseBtn>
    </div>

    <!-- <VirtualKeyboard
      :game-ended="isWordSolved || maxErrorsExceeded"
      :disabled-chars="charsClicked"
      @char-pressed="charPressed"
    /> -->

    <div class="keyboard-content"> <!-- the alphabet-->
      <kbd
        v-for="(char, index) of characters"
        :key="index"
        :class="{
          disabled:
            charsClicked.includes(char) || isWordSolved || maxErrorsExceeded
        }"
        @click="charPressed(char)"
      >
        <em>{{ char }}</em>
      </kbd>
    </div>

    <div class="flex">
      <p class="dialog__description" v-text="character"></p>
      <!-- <p class="dialog__description" v-text="accuracy"></p> -->
    </div>

    <BaseBtn class="reveal-word-btn" @click="revealTheWord">
      Reveal the word
    </BaseBtn>
  </div>
</template>

<script>
import randomWord from "random-words";
import { randomNumber, randomIndexesFromWord } from "@/shared/utils";
import { wordContainsArrayOfChars } from "@/shared/utils";
import { stopSound, playSound } from "@/shared/game-sounds";
import { errors } from "@/shared/errors-observable";
import Dialog from "@/components/Dialog";
//import guessWordSharedCode from "@/shared/guess-word-shared-code";
import BaseBtn from "@/shared/components/BaseBtn";

// Components
import DisplayWordChars from "@/shared/components/DisplayWordChars";
//import VirtualKeyboard from "@/shared/components/VirtualKeyboard";
import { getKeyboardChars } from "@/shared/utils";


export default {
  // props: ["soundRecord", "isRecording"],
  components: {
    //VirtualKeyboard,
    DisplayWordChars,
    Dialog,
    BaseBtn
  },

  //mixins: [guessWordSharedCode],

  data: () => ({
    showDialog: true, // <Now to Play> button set to be open at the start of the page
    userMode: true,
    word: "",
    character: "Character: ",
    charsClicked: [],
    characters: getKeyboardChars()
  }),
  /* mounted() {
    this.soundRecord.onresult = evt => {
      for (let i = evt.results.length - 1; i < evt.results.length; i++) {
        const result = evt.results[i];

        if (result.isFinal) this.CheckForCommand(result);
      }
    };
  }, */

  computed: {
    chars() {
      const hiddenCharsCount = randomNumber(1, this.word.length - 2);
      const hiddenIndexes = randomIndexesFromWord(hiddenCharsCount, this.word);
      return this.word.split("").map((char, index) => ({
        hidden: hiddenIndexes.includes(index),
        value: char
      }));
    },
    errorsCount: {
      get() {
        return errors.count;
      },
      set(val) {
        errors.count = val;
      }
    },

    hiddenChars() {
      return this.chars.filter(char => char.hidden).map(char => char.value);
    },

    isWordSolved() {
      return wordContainsArrayOfChars(
        this.charsClicked.join(),
        this.hiddenChars
      );
    },
    maxErrorsExceeded() { 
      return errors.count > errors.maxErrors;
    }
  },

  created() {
    this.startGame();
  },
  watch: {
    isWordSolved(val) {
      val && !this.maxErrorsExceeded && playSound("won");
    },

    maxErrorsExceeded(val) {
      val && playSound("lost");
    }
  },
 /*  beforeDestroy() {
    this.$refs.wordWrapper.removeEventListener(
      "animationend",
      this.animationEnded
    );
    stopSound("lost");
    stopSound("won");
  }, */
  
  methods: {
    startGame() {
      stopSound("lost");
      stopSound("won");
      this.word = randomWord().toUpperCase();
      this.errorsCount = 0;
      this.charsClicked = [];
    },
    charPressed(char) {
      const charIsDisabled = () => this.charsClicked.includes(char);
      if (!(this.isWordSolved || this.maxErrorsExceeded) && !charIsDisabled()) {
        this.characterPressed(char);
      }
    },
    characterPressed(char) {
      if (!this.hiddenChars.includes(char)) {
        this.errorsCount++;
        this.triggerError();
        playSound("wrong");
      } else playSound("correct");
      this.charsClicked.push(char);
    },
    open() {
      this.showDialog = !this.showDialog;
    },
    cancel() {
      this.showDialog = false;
    },
    /*triggerError() {
      const el = this.$refs.wordWrapper;
      el.classList.add("error");
      el.addEventListener("animationend", this.animationEnded);
    },

    animationEnded() {
      this.$refs.wordWrapper.classList.remove("error");
    }, */

    revealTheWord() {
      this.hiddenChars.forEach(char => this.charsClicked.push(char));
    },
    CheckForCommand(result) {
      const t = result[0].transcript;
      this.character = "Character: " + t;

      if (result[0].confidence >= 0.3) {
        if (t.trim().includes("reveal the word")) {
          this.revealTheWord();
        }
        else if (t.trim().includes("how to play")) {
          this.showDialog=!this.showDialog;
        }
        else if(t.trim().includes("cancel")) {
          this.showDialog=false;
        }
        //isWordSolved || maxErrorsExceeded
        else if (
          t.trim().includes("new game") &&
          (this.isWordSolved || this.maxErrorsExceeded)
        ) {
          this.startGame();
        } else if (t.trim().length == 1) {
          this.charPressed(t.trim());
        }
      } else if (t.trim() == "are") {
        this.charPressed("r");
      } else if (t.trim() == "ji") {
        this.charPressed("g");
      }
      else if (t.trim() == "see") {
        this.charPressed("c");
      }
      else if (t.trim() == "ese") {
        this.charPressed("s");
      }
    }
  }
};
</script>

<style lang="scss" scoped>
@import "~@/sass/mixins";
@import "~@/sass/variables";
@include gameSharedStyle();

::v-deep .reveal-word-btn {
  position: fixed;
  left: 20px;
  bottom: 20px;
}
.keyboard-content {
  display: grid;
  grid-template-columns: repeat(10, 2.3rem);
  gap: 8px;
  padding: 8px;
  border-radius: 5px;
  box-shadow: 2px 2px 3px 0, -1px -2px 8px 1px $hm_blue;

  > kbd {
    display: flex;
    align-items: center;
    justify-content: center;
    box-shadow: 0 0 3px -1px rgba(0, 0, 0, 0.3);
    border-radius: 5px;
    padding: 10px;
    cursor: pointer;
    user-select: none;

    &:nth-child(20) {
      grid-column: 1;
    }

    &.disabled {
      background: black;
      opacity: 0.3;
    }
  }
}
</style>

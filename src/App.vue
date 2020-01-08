<template>
  <div id="main-container">
    <nav-header @modeChanged="currentMode=$event"></nav-header>
    <keep-alive>
      <component :is="currentMode"></component>
    </keep-alive>
    <button class="button" @click="submitForm" :class="{'button-active': readyToSubmit}">Generate Passwords</button>
    <transition>
       <window v-show="displayWindow"></window>
    </transition>
  </div>
</template>

<script>
  import Header from "./components/Header.vue";
  import RegularModeForm from "./components/RegularModeForm.vue";
  import QuickModeForm from "./components/QuickModeForm.vue";
  import Window from "./components/Window.vue";
  import {EventBus} from "./main.js";

  export default {
    data: function() {
      return {
        displayWindow: false,
        generatedPasswords: [],
        currentMode: "regular-form",
        optionSelected: false,
        selectedData: {
          letters: {
            selected: false,
            number: 0,
            case: ""
          },
          numbers: {
            selected: false,
            number: 0
          },
          characters: {
            selected: false,
            number: 0
          },
          order: {
            selected: false,
            representation: "",
            description: ""
          }
        },
        randomNumber: 0 
      }
    },
    computed: {
      readyToSubmit: function() {
        if (this.currentMode === "regular-form") {
          return this.optionSelected;
        } else {
          return Boolean(this.randomNumber);
        }
      }
    },
    methods: {
      submitForm: function() {
        if (this.currentMode === "regular-form") {
          if (this.optionSelected) {
            let error = 0;
            let selectedCounter = 0
            for (let field in this.selectedData) {
              if (this.selectedData[field].selected) {
                selectedCounter++
              }
            } 
            if (this.selectedData.letters.selected && !this.selectedData.letters.number ||
                this.selectedData.letters.selected && !this.selectedData.letters.case) {
              EventBus.$emit("missedField", "letters");
              error++; 
            }
            if (this.selectedData.numbers.selected && !this.selectedData.numbers.number) {
              EventBus.$emit("missedField", "numbers");
              error++;
            }
            if (this.selectedData.characters.selected && !this.selectedData.characters.number) {
              EventBus.$emit("missedField", "special-characters");
              error++;
            }
            if (!this.selectedData.order.selected && selectedCounter >= 2) {
              EventBus.$emit("missedField", "order");
              error++
            }
            if (error === 0) {
              this.generatedPasswords = [];
              this.displayWindow = true;
              for (let x = 0; x <= 4; x++) {
                this.generatedPasswords.push(this.createSpecificPassword());
              }
              setTimeout(() => EventBus.$emit("passwordsCreated", this.generatedPasswords), 3000);
            }
          }
        } else {
          if (this.randomNumber === 0) {
            EventBus.$emit("missedField", "wildCard");
          } else {
              this.generatedPasswords = [];
              this.displayWindow = true;
              for (let x = 0; x <= 4; x++) {
                this.generatedPasswords.push(this.createRandomPassword());
              }
              setTimeout(() => EventBus.$emit("passwordsCreated", this.generatedPasswords), 3000);
          }

        }
      },
      createSpecificPassword: function() {
        let randomLetters = [];
        let randomNumbers = [];
        let randomCharacters = [];
        if (this.selectedData.letters.selected) {
           for (let i = 1; i <= this.selectedData.letters.number; i++) {
            let randomLetter = String.fromCharCode(65+Math.floor(Math.random() * 26));
            if (this.selectedData.letters.case === "capital") {
              randomLetters.push(randomLetter);
            } else if (this.selectedData.letters.case === "lowercase") {
              randomLetters.push(randomLetter.toLowerCase());
            } else {
              if (Math.round(Math.random())) {
                randomLetters.push(randomLetter);
              } else {
                randomLetters.push(randomLetter.toLowerCase());
              }
            }
          }
        }
        if (this.selectedData.numbers.selected) {
          for (let i = 1; i <= this.selectedData.numbers.number; i++) {
            let randomNumber = Math.floor(Math.random() * 10);
            randomNumbers.push(randomNumber);
          }  
        }
        if (this.selectedData.characters.selected) {
          for (let i = 1; i <= this.selectedData.characters.number; i++) {
            let randomCharacter = ["!", "@", "#", "$", "%", "^", "&", "*", "(", ")"][Math.floor(Math.random() * 10)];
            randomCharacters.push(randomCharacter);
          } 
        }
        if (randomLetters.length > 0 && randomNumbers.length === 0 && randomCharacters.length === 0) {
          return randomLetters.join("");
        } else if (randomLetters.length === 0 && randomNumbers.length > 0 && randomCharacters.length === 0) {
          return randomNumbers.join("");
        } else if (randomLetters.length === 0 && randomNumbers.length === 0 && randomCharacters.length > 0) {
          return randomCharacters.join("");
        } else {
          let passwordArray = [];
          if (this.selectedData.order.description.includes("Random")) {
            while (randomLetters.length || randomNumbers.length || randomCharacters.length) {
              let arrayPicker = Math.round(Math.random() * 2);
                if (arrayPicker === 0) {
                  if (randomLetters.length) {
                    passwordArray.push(randomLetters.pop());
                  }
                } else if (arrayPicker === 1) {
                  if (randomNumbers.length) {
                    passwordArray.push(randomNumbers.pop())
                  }
                } else {
                  if (randomCharacters.length) {
                    passwordArray.push(randomCharacters.pop());
                  }
                }
            }
          } else {

            if (this.selectedData.order.representation[0] === "a") {
              passwordArray = passwordArray.concat(randomLetters);
            } else if (this.selectedData.order.representation[0] === "1") {
              passwordArray = passwordArray.concat(randomNumbers);
            } else {
              passwordArray = passwordArray.concat(randomCharacters);
            }

            if (this.selectedData.order.representation[3] === "a") {
              passwordArray = passwordArray.concat(randomLetters);
            } else if (this.selectedData.order.representation[3] === "1") {
              passwordArray = passwordArray.concat(randomNumbers);
            } else {
              passwordArray = passwordArray.concat(randomCharacters);
            }

            if (this.selectedData.order.representation[6]) {
              if (this.selectedData.order.representation[6] === "a") {
                passwordArray = passwordArray.concat(randomLetters);
              } else if (this.selectedData.order.representation[6] === "1") {
                passwordArray = passwordArray.concat(randomNumbers);
              } else {
                passwordArray = passwordArray.concat(randomCharacters);
              }
            }
          }
          return passwordArray.join("");
        }
      },
      createRandomPassword: function() {
        let passwordArray = [];
        for (let i = 0; i < this.randomNumber; i++) {
          let diceRoll = Math.floor(Math.random()*3);
          if (diceRoll === 0) {
            let letter = String.fromCharCode(65+Math.floor(Math.random() * 26));
            if (Math.round(Math.random())) {
              passwordArray.push(letter);
            } else {
              passwordArray.push(letter.toLowerCase());
            }
          } else if (diceRoll === 1) {
            let character = ["!", "@", "#", "$", "%", "^", "&", "*", "(", ")"][Math.floor(Math.random() * 10)];
            passwordArray.push(character);
          } else {
            passwordArray.push(Math.floor(Math.random() * 10));
          }
        }
        return passwordArray.join("");
      }
    },
    components: {
      "nav-header": Header,
      "regular-form": RegularModeForm,
      "quick-form": QuickModeForm,
      "window": Window
    }, 
    created: function() {
      EventBus.$on("regularFormUpdate", (obj) => {
        let counter = 0; 
        for (let prop in obj) {
          if (obj[prop]) {
            counter++
          }
        }
        if (counter > 0) {
          this.optionSelected = true;
        } else {
          this.optionSelected = false;
        }
      })
      EventBus.$on("checkboxChecked", (action, typeOfCheckbox) => {
        if (action) {
          this.selectedData[typeOfCheckbox].selected = true;  
        } else {
          this.selectedData[typeOfCheckbox].selected = false;  
        }
        this.selectedData.order.selected = false;
        EventBus.$emit("resetOrderCheckbox");
      })
      EventBus.$on("numberSelected", (value, typeOfCheckbox) => {
        this.selectedData[typeOfCheckbox].number = value;
      })
      EventBus.$on("caseSelected", (letterCase) => {
        this.selectedData.letters.case = letterCase;
      })
      EventBus.$on("orderSelected", (obj) => {
        this.selectedData.order.selected = true;
        this.selectedData.order.representation = obj.representation;
        this.selectedData.order.description = obj.description;
      })
      EventBus.$on("needMorePasswords", ()=> {
        this.generatedPasswords = [];
        if (this.currentMode === "regular-form") {
          for (let x = 0; x <= 4; x++) {
            this.generatedPasswords.push(this.createSpecificPassword());
          }
        } else {
          for (let x = 0; x <= 4; x++) {
            this.generatedPasswords.push(this.createRandomPassword());
          }
        }
        setTimeout(() => EventBus.$emit("passwordsCreated", this.generatedPasswords), 3000);
      })
      EventBus.$on("closeWindow", ()=> {
        this.displayWindow = false;
      })
      EventBus.$on("numberForRandomPassword", (number)=> {
        this.randomNumber = number;
      })
    }
  }
</script>

<style>
  #main-container {
    height: 635px;
    width: 600px;
    background-color: #ffffff;
    border-radius: 5px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: top;
    padding: 25px;
  }

  .button {
    background-color: #f9fbfd;
    color: #95aac9;
    width: 200px;
    height: 45px;
    border-radius: 5px;
    margin-top: auto;
  }

  .button-active {
    background-color: #0c9 !important;
    color: white !important;
  }


</style>

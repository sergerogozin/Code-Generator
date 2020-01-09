<template>
    <div class="regular-form">
        <header>What should your password include?</header>
        <alpha-option checkboxID="letters" amount="5" @optionSelected="registerSelection"></alpha-option>
        <non-alpha-option checkboxID="numbers" amount="6" @optionSelected="registerSelection"></non-alpha-option>
        <non-alpha-option checkboxID="special-characters" amount="4" @optionSelected="registerSelection"></non-alpha-option>
        <div class="order-container">
            <div class="order-title" v-show="numberOfOptions > 1"><span :class="{'order-title-error': errorTitle}">Pick a style</span></div>
            <div v-show="numberOfOptions > 1" class="order-container-row">
                <password-order-option :order="firstOrderOption" @click.native="orderSelected=firstOrderOption"></password-order-option>
                <password-order-option :order="secondOrderOption" @click.native="orderSelected=secondOrderOption"></password-order-option>
            </div>
            <div v-show="numberOfOptions > 1" class="order-container-row" :class="{'order-container-one-option': numberOfOptions === 2}">
                <password-order-option :order="thirdOrderOption"  @click.native="orderSelected=thirdOrderOption"></password-order-option>
                <password-order-option v-show="numberOfOptions === 3" :order="fourthOrderOption"  @click.native="orderSelected=fourthOrderOption"></password-order-option>
            </div>
            <div v-show="numberOfOptions === 3" class="order-container-row">
                <password-order-option :order="fifthOrderOption" @click.native="orderSelected=fifthOrderOption"></password-order-option>
                <password-order-option :order="sixthOrderOption" @click.native="orderSelected=sixthOrderOption"></password-order-option>
            </div>
            <div v-show="numberOfOptions === 3" class="order-container-row order-container-one-option">
                <password-order-option :order="seventhOrderOption" @click.native="orderSelected=seventhOrderOption"></password-order-option>
            </div>
        </div>
    </div>
</template>

<script>
    import NonAlphaOption from "./NonAlphaOption.vue";
    import AlphaOption from "./AlphaOption.vue";
    import PasswordOrderOption from "./PasswordOrderOption.vue";
    import {EventBus} from "../main.js";

    export default {
        data: function() {
            return {
                errorTitle: false, 
                orderSelected: "",
                selectedOptions: {
                    letters: false,
                    numbers: false,
                    "special-characters": false
                },
                firstOrderOption: {
                    representation: "",
                    description : ""
                },
                secondOrderOption: {
                    representation: "",
                    description : ""
                },
                thirdOrderOption: {
                    representation: "",
                    description : ""
                },
                fourthOrderOption: {
                    representation: "123!@#abc",
                    description : "Numbers, characters, then letters"
                },
                fifthOrderOption: {
                    representation: "!@#abc123",
                    description : "Characters, letters, then numbers"
                },
                sixthOrderOption: {
                    representation: "!@#123abc",
                    description : "Characters, numbers, then letters"
                },
                seventhOrderOption: {
                    representation: "a!23b@#1c",
                    description: "Random order"
                }, 
            }
        },
        methods: {
            registerSelection: function(value, type) {
                this.selectedOptions[type] = value;
                EventBus.$emit("regularFormUpdate", this.selectedOptions);
            }
        },
        computed: {
            numberOfOptions: function() {
                let x = 0;
                for (let option in this.selectedOptions) {
                    if (this.selectedOptions[option]) {
                        x++
                    }
                }
                return x;
            }
        },
        watch: {
            selectedOptions: {
                deep: true,
                handler: function() {
                    if (this.selectedOptions.letters && this.selectedOptions.numbers && this.selectedOptions["special-characters"]) {
                        this.firstOrderOption.representation = "abc123!@#";
                        this.firstOrderOption.description  = "Letters, numbers, then characters";
                        this.secondOrderOption.representation = "abc!@#123";
                        this.secondOrderOption.description  = "Letters, characters, then numbers";
                        this.thirdOrderOption.representation = "123abc!@#";
                        this.thirdOrderOption.description  = "Numbers, letters, then characters"
                    } else if (this.selectedOptions.letters && this.selectedOptions.numbers) {
                        this.firstOrderOption.representation = "abc123";
                        this.firstOrderOption.description  = "Letters, then numbers";
                        this.secondOrderOption.representation = "123abc";
                        this.secondOrderOption.description  = "Numbers, then letters";
                        this.thirdOrderOption.representation = "ab32c1";
                        this.thirdOrderOption.description  = "Random order";
                    } else if (this.selectedOptions.letters && this.selectedOptions["special-characters"]) {
                        this.firstOrderOption.representation = "abc!@#";
                        this.firstOrderOption.description  = "Letters, then special characters";
                        this.secondOrderOption.representation = "!@#abc";
                        this.secondOrderOption.description  = "Special characters, then letters";
                        this.thirdOrderOption.representation = "ab!@c#";
                        this.thirdOrderOption.description  = "Random order";    
                    } else if (this.selectedOptions.numbers && this.selectedOptions["special-characters"]) {
                        this.firstOrderOption.representation = "123!@#";
                        this.firstOrderOption.description  = "Numbers, then special characters";
                        this.secondOrderOption.representation = "!@#123";
                        this.secondOrderOption.description  = "Special characters, then numbers";
                        this.thirdOrderOption.representation = "!@31#2"
                        this.thirdOrderOption.description = "Random order";  
                    } 
                }
            }
        },
        components: {
            "alpha-option": AlphaOption,
            "non-alpha-option": NonAlphaOption,
            "password-order-option": PasswordOrderOption
        },
        created: function() {
            EventBus.$on("missedField", (message) => {
                if (message === "order") {
                    this.errorTitle = true;
                }
            })
            EventBus.$on("resetOrderCheckbox", ()=> {
                this.errorTitle = false;
            }), 
            EventBus.$on("orderSelected", ()=> {
                this.errorTitle = false;
            })
        }
    }
</script>

<style>
    .regular-form {
        width: 100%;
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    header {
        font-size: 1.575rem;
        margin-top: 25px;
        margin-bottom: 20px;
    }

    .order-title {
        width: 100%;
        text-align: center;
        margin: 10px 0 10px 0;
    }

    .order-title span {
        border-bottom: 1px solid transparent;
    }

    .order-title-error {
        color: red !important;
        border-bottom: 1px solid red !important;
    }

    .order-container {
        display: flex;
        flex-direction: column;
        width: 100%;
    }

    .order-container-row {
        display: flex;
        width: 100%;
        justify-content: space-between;
        margin-bottom: 10px;
    }

    .order-container-one-option {
        justify-content: center !important;
        margin-bottom: 0 !important;
    }
</style>
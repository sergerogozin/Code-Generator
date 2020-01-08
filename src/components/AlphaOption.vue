<template>
    <div class="option-container" :class="{'option-container-error': errorBorder}" @click="toggleOption">
        <input type="checkbox" :id="checkboxID" :name="checkboxID">
        <label class="main-label checkbox-clone" :for="checkboxID" :class="{'option-selected': optionSelected}"></label>
        <label class="main-label" :for="checkboxID">{{checkboxID | toUpperCase | split}}</label>
        <div class="phrase" v-show="optionSelected">
            <span class="break-span"></span>
            <p>I want </p>
            <select class="non-alpha-select" v-model="numberSelected">
                <option disabled>Select one</option>
                <option v-for="x of Number(amount)">{{x + 4}}</option>
            </select>
            <p> {{checkboxID | split | pluralOrNot(numberSelected)}}</p>
            <span class="break-span"></span>
            <div class="radio-container">
                <label class="radio">
                    <input id="capital" type="radio" name="case" value="capital">
                    <span for="capital" @click="caseSelected='capital'">Capital</span>
                </label>
                <label class="radio">
                    <input id="lowercase" type="radio" name="case" value="lowercase">
                    <span for="lowercase" @click="caseSelected='lowercase'">Lowercase</span>
                </label>
                <label class="radio">
                    <input id="both" type="radio" name="case" value="both">
                    <span for="both" @click="caseSelected='both'">Both</span>
                </label>
            </div>
        </div>
    </div>
</template>

<script>
    import {EventBus} from "../main.js";

    export default {
        data: function() {
            return {
                optionSelected: false,
                numberSelected: "Select one",
                caseSelected: "",
                optionType: this.checkboxID,
                errorBorder: false
            }
        },
        methods: {
            toggleOption: function(event) {
                if (event.target.classList.contains("option-container") ||
                    event.target.classList.contains("main-label")) {
                    if (this.optionSelected) {
                        this.optionSelected = false;
                        EventBus.$emit("checkboxChecked", false, this.checkboxID);
                    } else {
                        this.optionSelected = true;
                        EventBus.$emit("checkboxChecked", true, this.checkboxID);
                    }
                }
            },
            radioClicked: function(event) {
                console.log(event.target)
            }
        },
        watch: {
            optionSelected: function() {
                this.$emit("optionSelected", this.optionSelected, this.optionType);
            },
            numberSelected: function() {
                EventBus.$emit("numberSelected", this.numberSelected, this.checkboxID);
                if (this.caseSelected) {
                    this.errorBorder = false;
                }
            },
            caseSelected: function() {
                EventBus.$emit("caseSelected", this.caseSelected);
                if (this.numberSelected !== "Select one") {
                    this.errorBorder = false;
                }
            }
        },
        props: [
            "checkboxID",
            "amount"
        ],
        filters: {
            toLowerCase: function(word) {
                return word.toLowerCase();
            },
            toUpperCase: function(word) {
                return word[0].toUpperCase() + word.slice(1);
            },
            split: function(word) {
                return word.split("-").join(" ");
            },
            pluralOrNot: function(word, value) {
                if (value === "1") {
                    return word.slice(0, word.length-1);
                } else {
                    return word;
                }
            }
        }, 
        created: function() {
            EventBus.$on("missedField", (field) => {
                if (this.checkboxID === field) {
                   this.errorBorder = true;
                }
            })
            EventBus.$on("passedField", (field) => {
                if (this.checkboxID === field) {
                    this.errorBorder = false;
                }
            })
        }
    }
</script>

<style>
    .option-container {
        width: 100%;
        height: 42px;
        border: 1px solid #d3e0e9;
        cursor: pointer !important;
        padding: 8px 10px;
        margin: 0 10px 0 10px;
        box-sizing: border-box;
        display: flex;
        align-items: center;
        margin-bottom: .5rem;
        font-size: .9rem;
        border-radius: 5px;
    }

    .option-container-error {
        border: 1px solid red !important;    
    }

    .option-container * {
        cursor: pointer !important;
    }

    input[type="checkbox"] {
        display: none;
    }

    .checkbox-clone {
        position: relative;
        width: 12px;
        height: 12px;
        border: 1px solid #C8CCD4;
        border-radius: 3px;
        vertical-align: middle;
        transition: background-color .1s ease;
        margin-right: 7px;
    }

    .checkbox-clone::after {
        content: '';
        position: absolute;
        top: 0px;
        left: 4px;
        width: 2px !important;
        height: 8px !important;
        opacity: 1;
        transform: rotate(45deg) scale(0);
        border-right: 2px solid white;
        border-bottom: 2px solid white;
        transition: all .3s ease;
        transition-delay: .15s; 
    }

    .option-selected {
        border-color: transparent;
        background: #0c9;
        animation: appear .6s ease;
    }

    .option-selected::after {
        opacity: 1;
        transform: rotate(45deg) scale(1)
    } 
    
    .break-span {
        height: 8px;
        width: 8px;
        border-radius: 50%;
        background-color: #216484;
        display: inline-block;
        margin: 0 15px;
    }

    .non-alpha-select {
        margin: 0px 5px;
        border: none;
        border-bottom: 1px solid #d3e0e9;
        text-align: center;
    }

    .non-alpha-select:focus {
        outline: none;
    }

    .phrase {
        display: flex;
        align-items: center;
        flex: 1;
    }

    .phrase p {
        margin: 0;
    }

    .phrase select {
        height: 20px;
    }

    .radio-container {
        display: flex;
        justify-content: start-flex;
    }

    .radio {
        position: relative;
        margin-right: 30px;
    }

    .radio input {
        display: none;    
    }

    .radio span::before, .radio span::after {
        content: "";
        height: 13px;
        width: 13px;
        border-radius: 50%;
        position: absolute;
        right: -19px;
        top: 2px;
    }

    .radio span::before {
        background-color: #d3e0e9;
    }

    .radio span::after {
        background-color: white;
        transform: scale(.85);
        transition: .3s ease;
    }

    input[type=radio]:checked + span::before {
        background-color: #0c9;
    }

    input[type=radio]:checked + span::after {
        transform: scale(.4);
    }

    @keyframes appear {
        0% {
            transform: scale(1, 1);
        }
        30% {
           transform: scale(1.25, 0.75); 
        }
        40% {
            transform: scale(0.75, 1.25);    
        }
        50% {
            transform: scale(1.15, 0.85);    
        }
        65% {
            transform: scale(.95, 1.05);    
        }
        75% {
            transform: scale(1.05, .95);    
        }
        100% {
            transform: scale(1, 1)    
        }
    }
</style>
<template>
    <div class="option-container" :class="{'option-container-border': errorBorder}" @click="toggleOption">
        <input type="checkbox" :id="checkboxID" :name="checkboxID">
        <label :for="checkboxID" class="checkbox-clone" :class="{'option-selected': optionSelected}"></label>
        <label :for="checkboxID">{{checkboxID | toUpperCase | split}}</label>
        <div class="phrase" v-show="optionSelected">
            <span class="break-span"></span>
            <p>I want </p>
            <select class="non-alpha-select" v-model="numberSelected">
                <option disabled>Select one</option>
                <option v-for="x of Number(amount)">{{x}}</option>
            </select>
            <p> {{checkboxID | split | pluralOrNot(numberSelected)}}</p>
        </div>
    </div>
</template>

<script>
    import {EventBus} from "../main.js"

    export default {
        data: function() {
            return {
                optionSelected: false,
                optionType: this.checkboxID,
                numberSelected: "Select one",
                errorBorder: false
            }
        },
        computed: {
            correctedCheckboxID: function() {
                if (this.checkboxID.split("-").length===2) {
                    return this.checkboxID.split("-")[1]
                } else {
                    return this.checkboxID
                }
            },
        },
        methods: {
            toggleOption: function(event) {
                if (event.target.classList.contains("option-container") ||
                    event.target.localName === "label") {
                    if (this.optionSelected) {
                        this.optionSelected = false;
                        this.errorBorder = false;
                        EventBus.$emit("checkboxChecked", false, this.correctedCheckboxID);
                    } else {
                        this.optionSelected = true;
                        EventBus.$emit("checkboxChecked", true, this.correctedCheckboxID);
                    }
                }
                console.log(this.numberSelected);
            }
        },
        watch: {
            optionSelected: function() {
                this.$emit("optionSelected", this.optionSelected, this.optionType);
            },
            numberSelected: function() {
                EventBus.$emit("numberSelected", this.numberSelected, this.correctedCheckboxID);
                this.errorBorder = false;
            }
        },
        props: [
            "checkboxID",
            "amount",
            "error"
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

    .option-container-border {
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
    }

    .phrase p {
        margin: 0;
    }

    .phrase select {
        height: 20px;
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
<template>
    <div class="quick-form">
        <header>How long should your password be?</header>
        <div class="random-number-selector" :class="displayRedBorder">
            <p>I want the password to contain </p>
            <select class="non-alpha-select" v-model="numberOfRandomPasswords">
                <option disabled>Select one</option>
                <option v-for="x in 11">{{x + 8}}</option>
            </select>
            <p>characters.</p>
        </div>
        <p class="explanation">Quick Mode generates passwords using both uppercase and lowercase letters, numbers, and special characters arranged in random order.</p>
    </div>
</template>

<script>
    import {EventBus} from "../main.js";

    export default {
        data: function() {
            return {
                numberOfRandomPasswords: "Select one",
                displayRedBorder: {
                    "red-border": false
                }
            }
        },
        watch: {
            numberOfRandomPasswords: function() {
                EventBus.$emit("numberForRandomPassword", this.numberOfRandomPasswords);
            }
        }
    
}
</script>

<style>
    .quick-form {
        width: 100%;
    }

    .quick-form header {
        text-align: center;
    }

    .non-alpha-select {
        margin: 0px 5px;
        border: none;
        border-bottom: 1px solid #d3e0e9;
        text-align: center;
    } 

    .random-number-selector {
        width: 100%;
        height: 42px;
        border: 1px solid #d3e0e9;
        cursor: pointer !important;
        padding: 8px 10px;
        box-sizing: border-box;
        display: flex;
        align-items: center;
        margin-bottom: .5rem;
        font-size: .9rem;
        border-radius: 5px;
    }

    .explanation {
        color: #6e7da2!important;
        font-size: .9rem;
    }
</style>
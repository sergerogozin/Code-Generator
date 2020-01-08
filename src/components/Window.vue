<template>
    <div class="screen-container">
        <div class="fade-screen" @click="closeWindow">
        </div>
        <div class="window">
            <transition name="fade"> 
                <div v-show="displayCopiedSign" class="copied-sign">
                   <p>Copied to clipboard</p>
                </div>        
            </transition>
            <transition name="fade">
                <div class="animation" v-show="displayLoadingAnimation">
                    <div class="icon-holder">
                        <transition name="rotate">
                            <img src="../assets/big-gear.svg" class="big-icon">
                        </transition>
                        <transition name="rotateSmallGear"> 
                            <img src="../assets/small-gear.svg" class="small-icon">
                        </transition>
                    </div>
                    <div class="loading-phrase">
                        <span>Generating new passwords</span>
                    </div>
                </div>
            </transition>
            <transition name="fade">
                <div class="password-container" v-show="displayPasswords">
                    <div class="cancel-holder" @click="closeWindow">
                        <img src="../assets/cancel-icon-gray.svg" class="cancel-icon">
                    </div>
                    <p class="password-header">Your new passwords</p>
                    <template v-for="(password, index) of passwords">
                        <div class="password-entry" :key="index">
                            <p><span>{{index+1}})</span>{{password}}</p>
                            <img src="../assets/copy-icon.svg" @click="copyToClipboard(password)">
                        </div>
                    </template>
                    <button class="refresh-button" @click="refreshPasswords">Refresh</button>
                </div>
            </transition>
        </div>
    </div>
</template>

<script>
    import {EventBus} from "../main.js";

    export default {
        data: function() {
            return {
                displayLoadingAnimation: true,
                displayCopiedSign: false,
                displayPasswords: false,
                passwords: []
            }
        },
        methods: {
            copyToClipboard: function(password) {
                this.displayCopiedSign = true;
                let el = document.createElement('textarea');
                el.value = password;    
                document.body.appendChild(el);
                el.select();
                document.execCommand('copy');
                document.body.removeChild(el);
                setTimeout(() => {this.displayCopiedSign = false}, 3000);
            },
            refreshPasswords: function() {
                EventBus.$emit("needMorePasswords");
                this.displayPasswords = false;
                setTimeout(()=>this.displayLoadingAnimation = true, 800);
            },
            closeWindow: function() {
                EventBus.$emit("closeWindow");
                this.displayPasswords = false;
                setTimeout(()=>this.displayLoadingAnimation=true, 1000);
            }
        },
        props: [
            "listOfPasswords"
        ],
        created: function() {
            EventBus.$on("passwordsCreated", (passwordArray) => {
                this.passwords = passwordArray;
                this.displayLoadingAnimation = false;
                setTimeout(()=>this.displayPasswords = true, 800);
            })
        }         
    }
</script>

<style>
    .screen-container {
        position: fixed;
        top: 0;
        bottom: 0;
        right: 0;
        left: 0;
        z-index: 10;
    }

    .fade-screen {
        height: 100%;
        width: 100%;
        background-color: rgba(0, 0, 0, .3);
        z-index: 11;
    }

    .copied-sign {
        position: fixed;
        display: flex;
        justify-content: center;
        align-items: center;
        background-color: #0c9;
        height: 40px;
        width: 175px;
        top: 20px;
        left: 50%;
        margin-left: -85px; /* Negative half of width. */
        z-index: 12;
        border-radius: 5px;
        color: white;
        transition: 10s;
    }

    .copied-sign p {
        margin: 0;
    }

    .window {
        position: fixed;
        width: 500px;
        height: 350px;
        top: 50%;
        left: 50%;
        margin-top: -175px; /* Negative half of height. */
        margin-left: -250px; /* Negative half of width. */
        box-sizing: border-box;
        background-color: white;
        border: 2px solid #216484;
        border-radius: 5px;
        z-index: 12;
        padding: 25px;
        animation: scale .4s forwards;
    }

    .icon-holder {
        margin-top: 25px;
    }

    .loading-phrase {
        margin-top: 20px;
        font-size: 20px;
    }

    .password-container {
        display: flex;
        flex-direction: column !important;
        justify-content: flex-start;
        align-items: center;
        width: 100%;
        height: 100%;
    }

    .password-header {
        margin-bottom: 15px;
        font-size: 21px;
        color: black !important;
    }

    .cancel-holder {
        display: flex;
        justify-content: flex-end;
        width: 100%;
        cursor: pointer;
    }

    .cancel-icon {
        height: 10px;
    }

    .cancel-icon:hover {
        filter: brightness(1.05);
    }

    .password-entry {
        display: flex;
        justify-content: flex-start;
        width: 85%;
        border-radius: 5px;
        border: 1px solid #d3e0e9;
        margin-bottom: 10px;
        height: 30px;
    }

    .password-entry p {
        flex: 1;
        padding-left: 10px;
        font-size: .95em;
        position: relative;
        top: 1px;
    }

    .password-entry span {
        color: #d3e0e9;
        margin-right: 10px;
    }

    .password-entry img {
        height: 20px;
        border-left: 1px solid #d3e0e9;
        cursor: pointer;
        padding: 5px 10px;
    }

    .password-entry * {
        padding: 5px 0;
    }

    .refresh-button {
        background-color: #216484;
        color: white;
        margin: auto 0 0 0 !important;
    }

    .animation {
        height: 100%;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
    }

    .big-icon {
        height: 90px;
        animation: rotateBigGear 6s linear;
        animation-iteration-count: 3;
    }

    .small-icon {
        height: 68px;
        position: relative;
        top: -70px;
        right: 19px;
        animation: rotateSmallGear 6s linear;
        animation-iteration-count: 3;
    }

    .fade-enter-active, .fade-leave-active {
        transition: opacity .5s;
    }
    
    .fade-enter, .fade-leave-to {
        opacity: 0;
    }

    @keyframes rotateSmallGear {
        0% {
            transform: rotate(0deg);
        }

        100% {
            transform: rotate(360deg);
        }
    } 

    
    @keyframes rotateBigGear {
        0% {
            transform: rotate(0deg);
        }

        100% {
            transform: rotate(180deg);
        }
    } 

    @keyframes scale {
        0% {
            transform: scale(.3);
        }

        100% {
            transform: scale(1);
        }
    }

    @keyframes remove {
        0% {
            transform: scale(.3);
        }

        100% {
            transform: scale(1);
        }
    }
</style>
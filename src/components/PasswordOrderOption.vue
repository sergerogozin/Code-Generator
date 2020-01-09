<template>
    <div class="order-box">
        <label class="order-radio" @click="updateRadioStatus">
            <input :id="order.representation" type="radio" name="orderOption" :key="order.representation" v-model="radioSelected">
            <div :for="order.representation">
                <p>{{order.representation}}</p>
                <p>{{order.description}}</p> 
            </div>
        </label>
    </div>
</template>

<script>
    import {EventBus} from "../main.js";

    export default {
        data: function() {
            return {
                radioSelected: false
            }
        },
        methods: {
            updateRadioStatus: function() {
                this.radioSelected = "checked";
                EventBus.$emit("orderSelected", this.order);
            }
        },
        props: [
            "order"
        ],
        created: function() {
            EventBus.$on("resetOrderCheckbox", ()=> {
                this.radioSelected = false;
            })
        }
    }
</script>

<style>
    .order-box {
        border: 1px solid #d3e0e9;
        cursor: pointer;
        border-radius: 5px;
        font-size: .9rem;
    }

    .order-box * {
        cursor: pointer !important;
    }

    .order-radio {
        position: relative;
    }

    .order-radio input {
        display: none;    
    }

    .order-radio div {
        height: 100%;
        padding: 8px 10px 8px 28px;
        height: 56px;
        width: 265px !important;
        box-sizing: border-box;
    }

    .order-radio div::before, .order-radio div::after {
        content: "";
        height: 15px;
        width: 15px;
        border-radius: 50%;
        position: absolute;
        left: 7px;
        top: 20px;
    }

    .order-radio div::before {
        background-color: #d3e0e9;
    }

    .order-radio div::after {
        background-color: white;
        transform: scale(.85);
        transition: .3s ease;
    }

    input[type=radio]:checked + div::before {
        background-color: #0c9;
    }

    input[type=radio]:checked + div::after {
        transform: scale(.4);
    }

    p {
        margin: 0;
    }

    p:nth-child(2) {
        margin-top: 5px;
        color: #6e7da2;
    }
</style>
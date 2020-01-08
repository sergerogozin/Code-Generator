<template>
    <div id="button-container">
        <button :class="{'button-clicked': leftButtonClicked}" @click="modeSelected">Regular Mode</button>
        <button :class="{'button-clicked': rightButtonClicked}" @click="modeSelected">Quick Mode</button>
    </div>
</template>

<script>

export default {
    data: function() {
        return {
            leftButtonClicked: true,
            rightButtonClicked: false,
            selectedForm: "regular-form"
        }
    },
    methods: {
        modeSelected: function(event) {
            if (event.target.innerText === "Regular Mode") {
                this.leftButtonClicked = true;
                this.rightButtonClicked = false;
                this.selectedForm = "regular-form";
            } else {
                this.leftButtonClicked = false;
                this.rightButtonClicked = true;
                this.selectedForm = "quick-form";
            }
        }
    },
    watch: {
        selectedForm: function() {
            this.$emit("modeChanged", this.selectedForm);
        }
    }
}
</script>

<style>
    #button-container {
        display: flex;
    }

    button {
        padding: 8px 11px;
        width: 150px;
        cursor: pointer;
        border: none;
        border-radius: 5px;
        background-color: white;
        font-size: 14.5px;
        font-weight: 500;
    }

    button:focus {
        outline: none;
    }

    button:nth-child(1) {
        margin-right: 10px;
    }

    button:nth-child(2) {
        margin-left: 10px;
    }

    .button-clicked {
        background-color: #216484;
        color: white;
    }
</style>
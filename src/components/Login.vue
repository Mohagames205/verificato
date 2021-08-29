<template>
<v-container>
  <v-stepper
    v-model="e6"
    vertical
  >

    <form name="username">
    <v-stepper-step
      :complete="e6 > 1"
      step="1"
    >
      Vul je Minecraft naam in
    </v-stepper-step>

    <v-stepper-content step="1">

      <v-alert 
        v-if="toggleError"
        dense
        type="error"
      >
      {{ errorMessage }}
      </v-alert>
    
    <v-text-field label="Minecraft name" :rules="rules" v-model="username"></v-text-field>
    <p>{{ username }} </p>
      <v-btn
        color="primary"
        @click="submitName()"
      >
        Continue
      </v-btn>
    </v-stepper-content>
    </form>

    <v-stepper-step
      :complete="e6 > 2"
      step="2"
    >
      Verifieer dat jij het bent
    </v-stepper-step>

    <v-stepper-content step="2">

      <v-alert
      dense
      type="info"
    >

    Voer <b>/link</b> uit ingame en vul de code hier in die je ziet.

    </v-alert>

    <v-alert
      v-if="codeIsWrong"
      dense
      type="error"
    >

      De code is ongeldig

    </v-alert>

    <v-text-field label="verificatiecode" v-model="verificationCode" />
      
      <v-btn
        color="primary"
        @click="verify()"
      >
        Verifieer
      </v-btn>
      <v-btn 
        text
        @click="e6 -= 1"
      >
        Cancel
      </v-btn>
    </v-stepper-content>

    <v-stepper-step
      :complete="e6 > 3"
      step="3"
    >
      Omleiden naar website
    </v-stepper-step>

    <v-stepper-content step="3">

      <v-alert
      dense
      type="success"
    >

      De code is geldig en u wordt omgeleid naar de website

    </v-alert>
    </v-stepper-content>
  </v-stepper>
</v-container>
</template>

<script>
 
  import axios from 'axios'

  export default {
    name: "Login",
    
    rules: [
        value => !!value || 'Required.',
        value => (value && value.length >= 3) || 'Min 3 characters',
      ],

    methods:  {

      submitName: function() {
        if(this.username.length != 0) {
            this.toggleError = false;
            axios.get("http://localhost:8000/api/verify?username=" + this.username)
            .then(() => {this.e6 = 2})
            .catch((error) => {
              this.error("Er is iets misgegaan: " + error);
            });
        }
        else {
          this.error("Gelieve een geldige Minecraft naam in te geven")
        }
      },

      verify: function() {
        axios.get(`http://localhost:8000/api/check?username=${this.username}&code=${this.verificationCode}`)
        .then((result) => {
          if(!result.data.isValid) {
            this.codeIsWrong = true;
            return;
          }

          this.e6 = 3;
          })
        .catch((error) => this.error("Er is iets misgegaan: " + error));
      },

      error: function(message) {
        this.toggleError = true;
        this.errorMessage = message;
      }
    },

    data () {
      return {
        rules: [],
        toggleError: false,
        e6: 1,
        username: "",
        errorMessage: "",
        verificationCode: undefined,
        codeIsWrong: undefined,
      }
    },
  }
</script>
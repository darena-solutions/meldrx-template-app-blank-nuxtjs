<template>
  <h1>Welcome</h1>
  <div>
    <button @click="login">Login</button>
  </div>
</template>

<script setup>
import {oauth2} from "fhirclient";

const config = useRuntimeConfig()
const {appBaseUrl, meldrxClientId, meldrxWorkspaceUrl} = config.public;

function login() {
  oauth2.authorize({
    clientId: meldrxClientId,
    scope: "openid profile fhirUser patient/Patient.read launch launch/patient",
    redirectUri: `${appBaseUrl}/login-callback`,
    iss: meldrxWorkspaceUrl,
  })

}

</script>
<template>
  <h1>Welcome</h1>
  <div>
    <button @click="login">Login</button>
  </div>
</template>

<script setup>
import {oauth2} from "fhirclient";

const config = useRuntimeConfig()
const {appBaseUrl, meldrxAuthUrl, meldrxClientId, meldrxWorkspaceId} = config.public;

function login() {
  oauth2.authorize({
    clientId: meldrxClientId,
    scope: "openid profile patient/*.read launch launch/patient",
    redirectUri: `${appBaseUrl}/login-callback`,
    iss: `${meldrxAuthUrl}/api/fhir/${meldrxWorkspaceId}`,
  })

}

</script>
<template>
  <Authenticated>

    <h1>Token</h1>

    <div v-if="state.token">
      <code v-text="state.token">
      </code>
    </div>
    <div v-else>
      Loading...
    </div>

  </Authenticated>
</template>

<script setup>
import fhirclient from "fhirclient";
import Authenticated from "~/layouts/authenticated.vue";

const state = reactive({'token': null})

onMounted(() => {
  return fhirclient.oauth2.ready()
    .then(client => state.token = client.getAuthorizationHeader().split(' ')[1])
})
</script>
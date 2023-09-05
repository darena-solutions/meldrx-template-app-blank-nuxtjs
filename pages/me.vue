<template>
  <Authenticated>

    <h1>Patient Information</h1>

    <ul v-if="state.patient">
      <li>
        <strong>id</strong> - {{ state.patient.id }}
      </li>
      <li>
        <strong>name</strong> - {{ state.patient.name[0].given.reduce((a, b) => a + ', ' + b) }}
        {{ state.patient.name[0].family }}
      </li>
      <li>
        <strong>date of birth</strong> - {{ state.patient.birthDate }}
      </li>
      <li>
        <strong>gender</strong> - {{ state.patient.gender }}
      </li>
      <li>
        <strong>address</strong> - {{
          state.patient.address.map(addr => `${addr.line.reduce((a, b) => a + ', ' + b)}, ${addr.city}, ${addr.state}, ${addr.postalCode}, ${addr.country}.`)[0]
        }}
      </li>
      <li>
        <strong>communication</strong> -
        {{ state.patient.communication.map(comms => comms.language.text).reduce((a, b) => a + ', ' + b) }}
      </li>
    </ul>
    <div v-else>
      Loading...
    </div>

  </Authenticated>
</template>

<script setup>
import fhirclient from "fhirclient";
import Authenticated from "~/layouts/authenticated.vue";

const state = reactive({'patient': null})

onMounted(() => {
  return fhirclient.oauth2.ready()
    .then(client => {

      const patientId = client.getPatientId();
      return client.request(`Patient?_id=${patientId}`, {flat: true});
    })
    .then(patients => state.patient = patients.length > 0 ? patients[0] : null)
})
</script>
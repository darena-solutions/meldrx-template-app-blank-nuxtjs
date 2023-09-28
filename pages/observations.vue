<template>
  <Authenticated>

    <h1>Patient Observations</h1>

    <hr>
    <h4>Add Height Observation</h4>
    <div>
      <label>Height:</label>
      <input v-model="state.heightForm.value"/>
    </div>
    <div>
      <button @click="createObservation">Submit</button>
    </div>
    <hr>

    <select v-model="state.category">
      <option v-for="cat in categories" :value="cat">{{ cat }}</option>
    </select>

    <div v-if="state.loading">
      Loading...
    </div>

    <ul v-if="state.observations">
      <li v-for="ob in state.observations">
        <template v-if="ob.valueCodeableConcept">{{ ob.code.text }} - {{ ob.valueCodeableConcept.text }}</template>
        <template v-else-if="ob.component">
          [{{ ob.effectiveDateTime }}] - {{ ob.code.text }}
          <ul>
            <li v-for="comp in ob.component">
              {{ comp.code.text }} - {{ comp.valueQuantity.value }}({{ comp.valueQuantity.unit }})
            </li>
          </ul>
        </template>
        <template v-else-if="ob.valueQuantity">
          [{{ ob.effectiveDateTime }}] {{ ob.code.text }} -  {{ ob.valueQuantity.value }}({{ ob.valueQuantity.unit }})
        </template>
        <template v-else>
          {{ ob.code.text }}
        </template>

      </li>
    </ul>

  </Authenticated>
</template>

<script setup>
import fhirclient from "fhirclient";
import Authenticated from "~/layouts/authenticated.vue";

const categories = [
  'social-history',
  'vital-signs',
  'imaging',
  'laboratory',
  'procedure',
  'survey',
  'exam',
  'therapy',
  'activity'
]

const state = reactive({
  loading: false,
  category: categories[0],
  observations: null,
  heightForm: {
    value: 0,
    unit: ['in', '[in_i]']
  }
})

function loadObservations() {
  if (state.loading) return;
  state.loading = true;
  state.observations = null
  return fhirclient.oauth2.ready()
    .then(client => {

      const patientId = client.getPatientId();
      return client.request(`Observation?subject=${encodeURIComponent('Patient/'+patientId)}&category=${state.category}`, {flat: true});
    })
    .then(observations => {
      console.log("Observations response", observations)
      state.observations = observations.filter(x => x.resourceType === 'Observation')
    })
    .finally(() => {
      state.loading = false;
      state.heightForm.value = 0.0;
    })
}

onMounted(loadObservations)
watch(() => state.category, loadObservations)

function createObservation() {
  if (state.loading) return;
  state.loading = true;

  return fhirclient.oauth2.ready()
    .then(client => {

      const patientId = client.getPatientId();

      client.create({
        "resourceType": "Observation",
        "meta": {
          "profile": ["http://hl7.org/fhir/StructureDefinition/vitalsigns"]
        },
        "status": "final",
        "category": [{
          "coding": [{
            "system": "http://terminology.hl7.org/CodeSystem/observation-category",
            "code": "vital-signs",
            "display": "Vital Signs"
          }],
          "text": "Vital Signs"
        }],
        "code": {
          "coding": [{
            "system": "http://loinc.org",
            "code": "8302-2",
            "display": "Body height"
          }],
          "text": "Body height"
        },
        "subject": {
          "reference": `Patient/${patientId}`
        },
        "effectiveDateTime": new Date().toISOString(),
        "valueQuantity": {
          "value": state.heightForm.value,
          "unit": state.heightForm.unit[0],
          "system": "http://unitsofmeasure.org",
          "code": state.heightForm.unit[1]
        }
      })
    })
    .finally(() => {
      state.loading = false;
      state.heightForm.value = 0.0;
    })
}

</script>
<template>
  <Authenticated>

    <h1>Patient Observations</h1>

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
          [{{ ob.effectiveDateTime }}] {{ ob.code.text }} - {{ ob.valueQuantity.value }}({{ ob.valueQuantity.unit }})
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
})

function loadObservations() {
  if (state.loading) return;
  state.loading = true;
  state.observations = null
  return fhirclient.oauth2.ready()
    .then(client => {

      const patientId = client.getPatientId();
      return client.request(`Observation?subject=${encodeURIComponent('Patient/' + patientId)}&category=${state.category}`);
    })
    .then(bundle => {
      console.log("Observations response", bundle)
      if (!bundle.entry){
        return;
      }

      if (bundle.entry.every(x => x.resource.resourceType === 'Bundle')) {
        state.observations = bundle.entry
          .map(x => 'entry' in x.resource ? x.resource.entry : [])
          .flat()
          .map(x => x.resource)
          .filter(x => x.resourceType === 'Observation')
      } else {
        state.observations = bundle.entry.map(x => x.resource).filter(x => x.resourceType === 'Observation')
      }
    })
    .finally(() => {
      state.loading = false;
      state.heightForm.value = 0.0;
    })
}

onMounted(loadObservations)
watch(() => state.category, loadObservations)

</script>
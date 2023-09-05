<template>
  Processing...
</template>

<script setup>
import {oauth2} from "fhirclient";

const router = useRouter()

onMounted(() =>
  oauth2
    .ready()
    .then(c => router.push("/me"))
)

</script>
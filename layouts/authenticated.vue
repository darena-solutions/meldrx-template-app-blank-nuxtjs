<template>
  <div>
    <div>
      <NuxtLink style="margin-left: 1rem; font-size: 2rem;" v-for="link in links" :href="link.to">{{ link.text }}</NuxtLink>
    </div>
    <slot/>
  </div>
</template>

<script setup>
const links = [
  {text: 'patient information', to: '/me'},
  {text: 'observations', to: '/observations'},
  {text: 'token', to: '/token'},
]
</script>
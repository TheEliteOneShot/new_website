<script setup lang="ts">
import { useHead } from '@vueuse/head'
import { useViewWrapper } from '/@src/stores/viewWrapper'

const viewWrapper = useViewWrapper()
viewWrapper.setPageTitle('Profile Edit Experience')

useHead({
  title: 'Profile Edit Experience',
})
</script>

<template>
  <EditProfileExperience />
</template>

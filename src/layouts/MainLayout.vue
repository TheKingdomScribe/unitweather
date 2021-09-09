<template>
  <q-layout view="lHh Lpr lFf">

    
    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>

const linksList = [
  {
    title: 'Docs',
    caption: 'quasar.dev',
    icon: 'school',
    link: 'https://quasar.dev'
  },
];

import { defineComponent, ref } from 'vue'

export default defineComponent({
  name: 'MainLayout',

  setup () {

    return {

    }
  }
})
</script>

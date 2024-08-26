<script lang="ts" setup>
import VerticalNavLink from '@layouts/components/VerticalNavLink.vue';
</script>
<template>
  <VerticalNavLink
      :item="{
        title: 'Dashboard',
        to: '/dashboard',
      }"
     
    />
  <VerticalNavLink
 
    :item="{
      title: 'Progress',
      icon: 'ri-table-alt-line',
      to: '/tables',
      
    }"
    
  />

  <VerticalNavLink
 
    :item="{
      title: 'Client Table',
      icon: 'ri-table-alt-line',
      to: '/clientable',
      
    }"
    
  />  
</template>



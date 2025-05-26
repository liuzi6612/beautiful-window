<template>
  <div class="text-center py-4">
    Copyright © 2024-{{ year }} xiejiahe, All Rights Reserved
  </div>
</template>

<script setup>
const now = new Date()
const year = now.getFullYear()
</script>

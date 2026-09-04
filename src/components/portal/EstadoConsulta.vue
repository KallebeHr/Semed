<template>
  <p v-if="consulta.carregando.value" role="status" class="p-alert">
    Carregando informações…
  </p>
  <div v-else-if="consulta.erro.value" class="p-alert error" role="alert">
    <p>{{ consulta.erro.value }}</p>
    <button class="p-button" @click="consulta.recarregar">
      Tentar novamente
    </button>
  </div>
</template>
<script setup>
defineProps({ consulta: { type: Object, required: true } });
</script>

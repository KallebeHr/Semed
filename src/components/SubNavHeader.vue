<template>
  <div class="portal-access">
    <div class="p-container access-inner">
      <span>Secretaria Municipal de Educação · Pedro II</span>
      <div class="access-tools">
        <router-link to="/acessibilidade">Acessibilidade</router-link
        ><button
          aria-label="Diminuir tamanho da fonte"
          :disabled="escala <= 90"
          @click="alterar(-10)"
        >
          A−</button
        ><button aria-label="Restaurar tamanho da fonte" @click="definir(100)">
          A</button
        ><button
          aria-label="Aumentar tamanho da fonte"
          :disabled="escala >= 200"
          @click="alterar(10)"
        >
          A+</button
        ><button :aria-pressed="contraste" @click="alternar">
          {{ contraste ? "Contraste padrão" : "Alto contraste" }}
        </button>
      </div>
      <span class="sr-only" role="status">Tamanho da fonte: {{ escala }}%</span>
    </div>
  </div>
</template>
<script setup>
import { ref, onMounted } from "vue";
const escala = ref(100),
  contraste = ref(false);
function salvar(k, v) {
  try {
    localStorage.setItem(k, v);
  } catch {
    /* Preferência ainda funciona nesta sessão. */
  }
}
function definir(v) {
  escala.value = Math.min(200, Math.max(90, Number(v) || 100));
  document.documentElement.style.fontSize = escala.value + "%";
  salvar("site-font-scale", String(escala.value));
}
function alterar(v) {
  definir(escala.value + v);
}
function alternar() {
  contraste.value = !contraste.value;
  document.documentElement.classList.toggle("high-contrast", contraste.value);
  salvar("site-high-contrast", String(contraste.value));
}
onMounted(() => {
  try {
    definir(localStorage.getItem("site-font-scale"));
    contraste.value = localStorage.getItem("site-high-contrast") === "true";
    document.documentElement.classList.toggle("high-contrast", contraste.value);
  } catch {
    /* Preferências indisponíveis neste navegador. */
  }
});
</script>
<style scoped>
.portal-access {
  border-bottom: 1px solid var(--p-border);
  background: var(--p-surface);
  font-size: 0.75rem;
}
.access-inner,
.access-tools {
  display: flex;
  gap: 0.85rem;
  align-items: center;
  flex-wrap: wrap;
}
.access-inner {
  justify-content: space-between;
  padding-block: 0.45rem;
}
.access-tools {
  gap: 0.3rem;
}
.access-tools a {
  margin-right: 0.5rem;
  color: var(--p-ink);
}
.access-tools button {
  padding: 0.25rem 0.5rem;
  min-height: 32px;
  border-radius: 0.3rem;
}
.access-tools button:hover {
  background: var(--p-soft);
}
@media (max-width: 720px) {
  .access-inner > span:first-child {
    display: none;
  }
  .access-inner {
    justify-content: center;
  }
}
</style>

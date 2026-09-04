<template>
  <div v-if="usuario" class="admin-layout">
    <aside class="admin-sidebar">
      <router-link class="admin-brand" to="/administracao"
        ><span class="p-eyebrow">SEDUC · PEDRO II</span
        ><strong>Administração</strong></router-link
      >
      <p class="admin-name">
        {{ usuario.nome }}<br /><span>{{
          CARGOS[cargoAtual(usuario.papel)]
        }}</span>
      </p>
      <nav aria-label="Administração">
        <router-link to="/administracao">Visão geral</router-link
        ><router-link v-for="n in menu" :key="n.path" :to="n.path">{{
          n.nome
        }}</router-link>
      </nav>
      <div class="admin-bottom">
        <router-link to="/">Abrir portal público ↗</router-link
        ><button class="p-button" @click="sair">Sair da conta</button>
      </div>
    </aside>
    <div class="admin-content">
      <p
        v-if="manutencao.dados.value[0]?.bloqueado"
        class="p-alert"
        role="status"
      >
        Manutenção em andamento. Os dados continuam disponíveis para consulta;
        aguarde a liberação antes de salvar alterações.
      </p>
      <router-view
        :key="
          route.path + usuario.papel + JSON.stringify(usuario.escolasVinculadas)
        "
      />
    </div>
  </div>
  <div v-else class="p-container p-page">
    <p role="status">Verificando acesso…</p>
  </div>
</template>
<script setup>
import { computed, watch } from "vue";
import { doc } from "firebase/firestore";
import { db } from "../../firebase";
import { useColecao } from "../../composables/useColecao";
import { useRouter, useRoute } from "vue-router";
import { useAuth } from "../../composables/useAuth";
import { CARGOS, cargoAtual } from "../../portal/permissoes";
const auth = useAuth(),
  { usuario, pode } = auth,
  router = useRouter(),
  route = useRoute();
const manutencao = useColecao(
  () => (usuario.value ? doc(db, "operacao", "estado") : null),
  () => usuario.value?.uid,
);
const links = [
  ["conteudo", "Conteúdos e serviços", "conteudos"],
  ["configuracao", "Página inicial e contatos", "configuracao"],
  ["escolas", "Escolas e publicação", "escolas"],
  ["merenda", "Alimentação escolar", "merenda"],
  ["nutricao", "Cardápios mensais", "cardapios"],
  ["nutricao", "Publicar vistorias", "vistorias"],
  ["atendimentos", "Solicitações", "solicitacoes"],
  ["alunos", "Alunos e vínculos", "alunos"],
  ["notas", "Notas e frequência", "notas"],
  ["usuarios", "Usuários e cargos", "usuarios"],
  ["auditoria", "Auditoria do sistema", "auditoria"],
  ["recuperacao", "Recuperação e ambiente", "recuperacao"],
];
const menu = computed(() =>
  links
    .filter((n) => pode(n[0]) && !(n[2] === "cardapios" && pode("conteudo")))
    .map((n) => ({ nome: n[1], path: "/administracao/" + n[2] })),
);
async function sair() {
  await auth.sair();
  await router.replace("/login");
}
watch(
  () => [
    usuario.value?.papel,
    usuario.value?.ativo,
    usuario.value?.uid,
    JSON.stringify(usuario.value?.escolasVinculadas),
  ],
  () => {
    if (!usuario.value) router.replace("/login");
    else if (route.meta.permissao && !pode(route.meta.permissao))
      router.replace("/administracao");
  },
);
</script>
<style scoped>
.admin-layout {
  display: grid;
  grid-template-columns: 250px minmax(0, 1fr);
  min-height: 100vh;
}
.admin-sidebar {
  background: var(--p-surface);
  padding: 2rem 1.2rem;
  border-right: 1px solid var(--p-border);
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}
.admin-brand {
  text-decoration: none;
  display: grid;
  gap: 0.4rem;
  color: var(--p-ink);
}
.admin-brand strong {
  font-size: 1.35rem;
}
.admin-name {
  font-size: 0.9rem;
}
.admin-name span {
  color: var(--p-teal);
  font-size: 0.8rem;
}
.admin-sidebar nav {
  display: grid;
  gap: 0.2rem;
}
.admin-sidebar nav a {
  padding: 0.65rem 0.8rem;
  text-decoration: none;
  border-radius: 0.5rem;
  color: var(--p-muted);
  font-size: 0.87rem;
  font-weight: 600;
}
.admin-sidebar nav .router-link-exact-active {
  background: var(--p-soft);
  color: var(--p-teal);
}
.admin-bottom {
  margin-top: auto;
  display: grid;
  gap: 0.8rem;
  font-size: 0.85rem;
}
.admin-content {
  padding: 2.5rem;
  min-width: 0;
}
.admin-content :deep(h1) {
  font-size: 2rem;
}
@media (max-width: 1000px) {
  .admin-layout {
    grid-template-columns: 1fr;
  }
  .admin-sidebar {
    padding: 1rem;
    border-right: 0;
    border-bottom: 1px solid var(--p-border);
    gap: 0.8rem;
  }
  .admin-sidebar nav {
    display: flex;
    flex-wrap: wrap;
  }
  .admin-brand,
  .admin-name {
    display: inline-block;
  }
  .admin-bottom {
    display: flex;
    align-items: center;
    gap: 1rem;
  }
  .admin-content {
    padding: 1.5rem 1rem;
  }
}
</style>

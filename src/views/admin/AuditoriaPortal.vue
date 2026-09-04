<template>
  <section>
    <header class="p-page-head">
      <h1>Auditoria do sistema</h1>
      <p>
        Últimas 200 ações da fonte selecionada. O histórico verificado é
        vinculado à alteração confirmada no banco.
      </p>
    </header>
    <label
      >Fonte do histórico
      <select v-model="fonte" class="p-input">
        <option value="auditoriaRegistros">
          Histórico verificado (versão 2)
        </option>
        <option value="auditoriaPortal">Portal anterior</option>
        <option value="auditoria">Alimentação anterior</option>
      </select>
    </label>
    <p v-if="fonte !== 'auditoriaRegistros'" class="p-alert">
      Registros anteriores foram preservados; não possuem a garantia de vínculo
      introduzida na versão 2.
    </p>
    <EstadoConsulta :consulta="consulta" />
    <div class="p-table-wrap">
      <table class="p-table">
        <thead>
          <tr>
            <th>Data</th>
            <th>Usuário</th>
            <th>Ação</th>
            <th>Registro</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="e in consulta.dados.value" :key="e.id">
            <td>{{ dataTexto(e.timestamp) }}</td>
            <td>{{ e.usuarioNome }}</td>
            <td>{{ e.acao.replaceAll("_", " ") }}</td>
            <td>
              {{ e.colecao || e.tipo }} · {{ e.documentoId }}
              <details v-if="e.versaoEsquema === 1">
                <summary>Conferir alteração</summary>
                <p>{{ e.caminho }}</p>
                <strong>Antes</strong>
                <pre>{{ JSON.stringify(e.dadosAntes, null, 2) }}</pre>
                <strong>Depois</strong>
                <pre>{{ JSON.stringify(e.dadosDepois, null, 2) }}</pre>
              </details>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </section>
</template>
<script setup>
import { ref } from "vue";
import { collection, query, orderBy, limit } from "firebase/firestore";
import { db } from "../../firebase";
import { useColecao } from "../../composables/useColecao";
import { dataTexto } from "../../portal/validacao";
import EstadoConsulta from "../../components/portal/EstadoConsulta.vue";
const fonte = ref("auditoriaRegistros");
const consulta = useColecao(
  () =>
    query(
      collection(db, fonte.value),
      orderBy("timestamp", "desc"),
      limit(200),
    ),
  fonte,
);
</script>

<style scoped>
pre {
  white-space: pre-wrap;
  overflow-wrap: anywhere;
  max-height: 22rem;
  overflow: auto;
  font-size: 0.8rem;
}
details {
  max-width: 42rem;
}
</style>

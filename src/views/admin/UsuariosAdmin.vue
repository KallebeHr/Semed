<template>
  <div class="p-stack">
    <header class="p-page-head">
      <p class="p-eyebrow">MASTER</p>
      <h1>Usuários e cargos</h1>
      <p>Defina as atribuições e as escolas de cada profissional.</p>
    </header>
    <div class="p-actions">
      <button class="p-button primary" @click="abrir()">
        + Cadastrar acesso</button
      ><button class="p-button" @click="copiarLink">
        Copiar link de acesso
      </button>
    </div>
    <p v-if="mensagem" role="status" class="p-alert success">{{ mensagem }}</p>
    <p v-if="erro" role="alert" class="p-alert error">{{ erro }}</p>
    <EstadoConsulta :consulta="contas" />
    <form v-if="editando" class="p-card p-stack" @submit.prevent="salvar">
      <h2>{{ existente ? "Editar acesso" : "Cadastrar acesso" }}</h2>
      <fieldset class="p-stack" :disabled="ocupado">
        <label v-if="!existente && !form.uid" class="p-check"
          ><input type="checkbox" v-model="criarConta" /> Criar também a conta
          de e-mail e senha</label
        >
        <label v-if="!criarConta || form.uid" class="p-field"
          >UID do Firebase Authentication<input
            aria-label="UID do Firebase Authentication"
            v-model="form.uid"
            :readonly="existente || uidCriado"
            required
            maxlength="128"
          /><small
            >O ID do perfil precisa ser o mesmo UID da conta. Criar um perfil
            não cria uma identidade no Authentication.</small
          ></label
        >
        <div class="p-grid two">
          <label class="p-field"
            >Nome completo<input
              aria-label="Nome completo"
              v-model="form.nome"
              required
              maxlength="160"
              autocomplete="name"
          /></label>
          <label class="p-field"
            >E-mail<input
              aria-label="E-mail"
              v-model="form.email"
              required
              type="email"
              maxlength="200"
              :readonly="existente || uidCriado"
              autocomplete="off"
          /></label>
          <label v-if="criarConta && !uidCriado" class="p-field"
            >Senha temporária<input
              aria-label="Senha temporária"
              v-model="form.senha"
              required
              type="password"
              minlength="12"
              autocomplete="new-password"
            /><small
              >Compartilhe a senha por um canal seguro. A pessoa pode usar
              “Esqueci minha senha” em /login.</small
            ></label
          >
          <label class="p-field"
            >Cargo<select aria-label="Cargo" v-model="form.papel" required>
              <option v-for="(c, id) in CARGOS" :key="id" :value="id">
                {{ c.nome || c }}
              </option>
            </select></label
          >
        </div>
        <label class="p-check"
          ><input type="checkbox" v-model="form.ativo" /> Acesso administrativo
          ativo</label
        >
        <fieldset class="p-stack">
          <legend>Escolas vinculadas</legend>
          <p class="p-small">
            Diretores e professores precisam de pelo menos uma escola. Limite de
            20 vínculos.
          </p>
          <EstadoConsulta :consulta="escolas" />
          <div class="p-grid two">
            <label v-for="e in escolas.dados.value" :key="e.id" class="p-check"
              ><input
                v-model="form.escolasVinculadas"
                type="checkbox"
                :value="e.id"
              />{{ e.nome }}{{ e.ativo === false ? " (inativa)" : "" }}</label
            >
          </div>
        </fieldset>
        <p v-if="existente" class="p-small">
          O e-mail de login é alterado no Firebase Authentication. A suspensão
          aqui remove o acesso da equipe; não exclui a conta.
        </p>
        <div class="p-actions">
          <button class="p-button primary">
            {{ ocupado ? "Salvando…" : "Salvar acesso" }}</button
          ><button type="button" class="p-button" @click="editando = false">
            Fechar
          </button>
        </div>
      </fieldset>
    </form>
    <label class="p-field"
      >Pesquisar usuário<input
        aria-label="Pesquisar usuário"
        v-model="termo"
        type="search"
    /></label>
    <div class="p-table-wrap">
      <table class="p-table">
        <caption class="sr-only">
          Contas da equipe
        </caption>
        <thead>
          <tr>
            <th>Nome / e-mail</th>
            <th>Cargo</th>
            <th>Situação</th>
            <th>Ação</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="u in filtradas" :key="u.id">
            <td>
              {{ u.nome }}<br /><span class="p-small">{{ u.email }}</span>
            </td>
            <td>{{ rotuloCargo(u.papel) }}</td>
            <td>{{ u.ativo === false ? "Suspenso" : "Ativo" }}</td>
            <td>
              <button class="p-button" @click="abrir(u)">Editar acesso</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <p v-if="!filtradas.length && !contas.carregando.value" class="p-empty">
      Nenhuma conta encontrada.
    </p>
  </div>
</template>
<script setup>
import { computed, reactive, ref } from "vue";
import { collection } from "firebase/firestore";
import { db } from "../../firebase";
import { useColecao } from "../../composables/useColecao";
import { CARGOS, cargoAtual } from "../../portal/permissoes";
import { salvarPerfil } from "../../portal/usuarios";
import { normalizarBusca, mensagemErro } from "../../portal/validacao";
import EstadoConsulta from "../../components/portal/EstadoConsulta.vue";
const rotuloCargo = (papel) => CARGOS[cargoAtual(papel)] || papel;
const contas = useColecao(() => collection(db, "usuarios")),
  escolas = useColecao(() => collection(db, "escolas"));
const form = reactive({}),
  editando = ref(false),
  existente = ref(false),
  criarConta = ref(false),
  uidCriado = ref(false),
  ocupado = ref(false),
  erro = ref(""),
  mensagem = ref(""),
  termo = ref("");
const filtradas = computed(() =>
  contas.dados.value
    .filter((u) =>
      normalizarBusca(u.nome + " " + u.email).includes(
        normalizarBusca(termo.value),
      ),
    )
    .sort((a, b) => a.nome.localeCompare(b.nome)),
);
function abrir(u) {
  if (ocupado.value) return;
  Object.assign(form, {
    uid: u?.id || "",
    nome: u?.nome || "",
    email: u?.email || "",
    papel: cargoAtual(u?.papel) || "professor",
    ativo: u?.ativo !== false,
    escolasVinculadas: [...(u?.escolasVinculadas || [])],
    senha: "",
  });
  existente.value = !!u;
  criarConta.value = false;
  uidCriado.value = false;
  editando.value = true;
  erro.value = "";
  mensagem.value = "";
}
async function salvar() {
  if (ocupado.value) return;
  ocupado.value = true;
  erro.value = "";
  mensagem.value = "";
  try {
    if (
      ["diretor", "professor"].includes(form.papel) &&
      !form.escolasVinculadas.length
    )
      throw new Error("Vincule ao menos uma escola ao diretor ou professor.");
    await salvarPerfil(form, criarConta.value && !form.uid);
    editando.value = false;
    form.senha = "";
    mensagem.value =
      "Acesso salvo. O profissional já pode entrar pelo link /login.";
  } catch (e) {
    uidCriado.value = criarConta.value && !!form.uid;
    if (uidCriado.value) {
      criarConta.value = false;
      form.senha = "";
    }
    erro.value =
      mensagemErro(e) +
      (uidCriado.value
        ? " A conta foi criada. O UID foi preservado; salve novamente para concluir o perfil."
        : "");
  } finally {
    ocupado.value = false;
  }
}
async function copiarLink() {
  try {
    await navigator.clipboard.writeText(location.origin + "/login");
    mensagem.value = "Link de acesso copiado.";
  } catch {
    erro.value = "Copie este link: " + location.origin + "/login";
  }
}
</script>
<style scoped>
fieldset {
  border: 0;
  min-width: 0;
}
</style>

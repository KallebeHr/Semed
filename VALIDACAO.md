# Validação — versão 2.0.0

Verificação local em 03/09/2026. Os testes usam projetos fictícios `demo-*` nos emuladores. Nenhuma conta, regra, índice ou informação de produção foi alterada.

| Verificação | Resultado |
| --- | --- |
| Build com cliente explícito | Concluído para `pedro-ii` / `recallapp-14074`. |
| Análise estática | Sem erros ou avisos de código. |
| Dependências de produção | `npm audit --omit=dev`: nenhuma vulnerabilidade conhecida reportada nesta execução; não é garantia de ausência de falhas. |
| Integração de composables, permissões, identificação e operações | 32 testes aprovados. |
| Regras reais do Firestore | 28 testes aprovados: 17 anteriores adaptados ao protocolo de auditoria e 11 regressões dos novos controles. |
| Recuperação e migração de notas | 10 testes aprovados, com Authentication e Firestore emulados. |
| Seleção e publicação por cliente | 3 testes aprovados. |
| Navegador — fluxos completos | Publicação, escola, serviço, cardápio/PDF, resumo público de vistoria, nova vistoria com CPF, cadastro de equipe, sessão do Master, auditoria e seção de recuperação verificados. |
| Navegador — professor e família | Nota/frequência, restrição de rota, solicitação, resposta e boletim verificados. |
| Navegador — acessibilidade e layout | Rotas públicas, busca/menu, contraste e fonte em 200%; sem transbordamento horizontal na largura de 390 px. |

São **73 testes automatizados aprovados**, além do roteiro de navegador. Os testes de negação usam o banco emulado com regras efetivas; não dependem apenas de ocultar botões. Os testes de integração simulam o SDK e complementam os testes das regras e do navegador.

Casos centrais: gravação sem auditoria; evento sem operação; estado anterior ou nome do autor forjado; tentativa de alterar histórico; nota com ID arbitrário; normalização Unicode; CPF inválido e nome divergente; estrutura e cálculo de checklist com oito itens; vínculos escolares e de família; alterações de cargo; manutenção; arquivo de backup alterado; senha errada; destino já ocupado; retomada com divergência; preservação dos documentos, credenciais e referências na restauração; decisão explícita sobre notas duplicadas.

O ensaio de recuperação incluiu alunos, notas, novos cardápios, solicitações, contas, custom claims, documentos com tipos Firebase e subcoleções sem documento pai. Um login por senha foi confirmado após restauração no Authentication emulado. O emulador utiliza hashes fictícios; o caminho de produção utiliza a importação de hashes do Admin SDK e precisa de ensaio com os parâmetros reais do projeto. A verificação local não comprova IAM, índices publicados ou configuração de autenticação de produção.

O build emite avisos de otimização sobre um módulo Firebase maior que 500 kB e importação dinâmica de autenticação já utilizada estaticamente. Não são erros de compilação; a redução desses pacotes permanece uma melhoria de desempenho. Nenhum desses avisos foi ocultado para declarar sucesso.

## Repetir a validação

Node 22.12+; Java 21 para os emuladores. Instale as dependências com `npm ci`.

```sh
npm run lint
npm run test:integracao
npm run test:regras
npm run test:seguranca
npm run test:recuperacao
npm run test:clientes
npx playwright install chromium
npm run test:navegador
npm run build -- --cliente pedro-ii
```

Execute as suítes de emuladores sequencialmente. Elas apagam apenas seus projetos fictícios para preparar os dados de teste. Não utilize credenciais ou registros reais nesses testes. As contas `example.test` e senhas de teste não existem no Firebase de produção.

## Limites da validação

Não foram realizados teste de invasão independente, carga de produção, certificação de acessibilidade, restauração com credenciais reais, teste do bucket legado ou verificação jurídica. Vistorias já publicadas e notas antigas do banco real não foram acessadas ou migradas. O protocolo de migração e revisão está em **OPERACAO-E-RECUPERACAO.md**. A liberação comercial deve considerar esses passos e a homologação da equipe do cliente.

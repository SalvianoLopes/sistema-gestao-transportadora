# CLAUDE.md — Murici Ipiranga

## Identificação
- **App real:** https://murici-ipiranga.vercel.app
- **Código:** `C:\Users\Desktop\portfolio\murici-ipiranga\index.html` (path de outra máquina/sessão); nesta máquina (Salviano, sessão 2026-08-11) foi clonado em `C:\Users\Salviano\Projetos\sistema-gestao-transportadora`
- **GitHub:** `git@github.com:SalvianoLopes/-sistema-gestao-transportadora.git`
- **Vercel project:** `prj_qiIMnKXbQeUqioLfXZYeI8fbhjLj` (project-xytnq)
- **Supabase:** `pneakxdlbyuysjorhjdu` (murici-ipiranga, sa-east-1)

## Deploy
Sempre usar:
```
cd C:\Users\Desktop\portfolio\murici-ipiranga
git add index.html
git commit -m "..."
git push origin main
vercel deploy --prod
```
O `.vercel/project.json` aponta para o projeto REAL (`prj_qiIMnKXbQeUqioLfXZYeI8fbhjLj`). Não alterar.

## Stack
- React 18 via CDN (sem build) — arquivo único `index.html`
- `const {createElement:h, useState, useEffect, useMemo} = React`
- Supabase via fetch nativo — usar `db.get()`, `db.post()`, `db.patch()` — NUNCA `fetch` direto
- Credenciais: `SURL` e `SKEY` (não `SUPABASE_URL`/`SUPABASE_ANON_KEY`)
- jsPDF + SheetJS (XLSX) via CDN

## Padrões de código
- Sem JSX — usar `h('div', {className:'...'}, ...)`
- CSS classes do design system: `card`, `sh`, `st`, `ss`, `kg`, `kc`, `kl`, `kv`, `tw`, `fg`, `lbl`, `r2`, `btn btn-r`, `btn btn-o`, `ov`, `mod`, `mh`, `mb`, `mf`
- Inputs: fundo branco `#fff`, texto preto `#111`
- Utilitários: `fmt()`, `R()`, `iso()`, `fd()`, `toast()`, `Ldg`, `Emp`, `Bs`

## Navegação (PAGES e NAV)
```js
const PAGES = {dashboard, entrada, liberacoes, descarga, frota, equipe, escala, recargas, ranking, despesas}
const NAV = [{k:'dashboard',...}, ..., {k:'escala', l:'📅 Escala'}, ...]
```

## Abas implementadas
| Aba | Tabela Supabase | Observação |
|---|---|---|
| Dashboard | entrada_diaria, vendas_diarias | KPIs mensais |
| Entrada do Dia | entrada_diaria | CRUD + importação Excel |
| Liberações | entrada_diaria | Visão do dia (padrão ao abrir) |
| Descarga FEMSA | descarga_lancamento, descarga_ref | Reembolso paletes |
| Frota | frota, manutencao_os, manutencao_itens | Cadastro (doc., motorista fixo) + gestão de manutenção |
| Equipe | equipe | CRUD colaboradores |
| **Escala** | **escala_rascunho** | Filtro por data, CRUD, export Excel |
| Recargas | entrada_diaria | Timeline + fechamento |
| Ranking | entrada_diaria | Score de incentivos |
| Despesas | despesas | Gastos operacionais |

## Aba Escala — detalhes
- Tabela: `escala_rascunho` (data, placa, tipo, motorista, mat_mot, aj1, mat_aj1, aj2, mat_aj2, num_transp, rota)
- Selects populados de `frota` (ativas) e `equipe` (ATIVO)
- Matrícula preenchida automaticamente ao selecionar motorista/ajudante
- Export Excel: todas as colunas incluindo Mat. Ajudante 1 e Mat. Ajudante 2
- Nome da aba Excel usa `dt` (YYYY-MM-DD), não `fd(dt)` (DD/MM/YYYY tem barra — XLSX rejeita)

## Recarga Manual
- Campos: `valor_recarga_manual_motorista`, `valor_recarga_manual_ajudante`
- Se preenchido: sobrescreve o valor padrão (VAN R$50/R$60, VUC R$70/R$60)
- Se vazio: usa valor padrão conforme tipo do veículo
- Status "Recarga" é distinto de "1ª Saída" e "PE"

## Regras de negócio importantes
- VAN: motorista R$50, ajudante R$50 (recarga padrão)
- VUC: motorista R$70, ajudante R$60 (recarga padrão)
- Ondas: 1ª (VAN 06:00-06:30), 2ª (VUC 06:30-07:00), 3ª (08:00)
- Venda total dia = vendas_diarias.venda_total + soma recargas individuais

## Aba Manutenção — detalhes

### Tabelas Supabase
- **frota**: (ver seção 5.2 no DOSSIE — desatualizado) agora também tem `marca`, `ano`, `venc_licenciamento`, `venc_seguro`. `tipo` deixou de ter CHECK (era só VAN/VUC, agora livre — curado na UI: VAN/VUC/TOCO/TRUCK/CARRETA/UTILITARIO/OUTRO). `status` aceita `INATIVO` além de ATIVO/MANUTENCAO/AGUARDANDO_PECA/LIBERADO — no formulário de Frota só ATIVO/INATIVO são selecionáveis manualmente, MANUTENCAO é automático (setado pelo fluxo de OS)
- **manutencao_os**: id, placa (FK → frota.placa, adicionada 2026-08-11), data_entrada, data_retorno, oficina, status (ABERTO/CONCLUIDO), km_entrada, km_retorno, observacoes, created_at, **motorista_id** (FK equipe — quem dirigia no momento da quebra, é o que conta no ranking de ofensores), **motorista_oficina_id** (FK equipe — quem levou o veículo até a oficina, opcional, só informativo — papel distinto do motorista_id), **previsao_saida** (date), **categoria_falha** (CHECK: Motor/Freios/Baia-Sider/Suspensão/Elétrica/Pneus/Troca de Óleo-Preventiva/Colisão-Mau uso), **valor_total** (numeric, editável tanto na abertura quanto no fechamento da OS, já que o custo real geralmente só é conhecido no retorno)
- **manutencao_itens**: id, os_id (FK → manutencao_os), placa, tipo (peca/servico), descricao, quantidade, data_troca, created_at
- Existe uma tabela legada `manutencao` (schema antigo, pré-refactor de maio/2026) com ~10 registros órfãos, não usada pelo app — deixada de fora do escopo intencionalmente

### Fluxo
1. "+ Novo Envio" → abre OS em manutencao_os (com motorista, previsão, categoria, valor opcional) + marca frota como MANUTENCAO
2. "📦 Itens / Peças" → modal para adicionar peças/serviços à OS (manutencao_itens)
3. "✓ Retornou" → fecha OS (status=CONCLUIDO), permite confirmar/ajustar valor_total final + marca frota como ATIVO

### Tabs da aba
- **Ativas**: cards com cor por criticidade (azul <10d, laranja 10-19d, amarelo 20-29d, vermelho 30d+) + KPIs
- **Histórico**: tabela expansível por linha — clica na linha para ver peças/serviços daquela OS
- **Análises**: 6 cards + gráfico de barras — ver abaixo

### Análises — como funciona
- Carrega todos os registros de manutencao_os + manutencao_itens + equipe em memória
- Agrupamento feito em JS (sem GROUP BY no banco)
- **Frequência de quebras por mês**: agrupa `data_entrada` por `YYYY-MM`, barras de `<div>` (sem lib de gráfico — projeto não tem canvas/chart lib)
- **Frequência por Veículo**: count de OS por placa + dias médios (só OS concluídas)
- **Maior Downtime**: ranking por soma total de dias parados por placa (não só média)
- **Maior Custo Acumulado**: ranking por soma de `valor_total` por placa
- **Ofensores por Motorista**: agrupa por `motorista_id` (quebra, não quem levou à oficina) → nome via `equipe`
- **Categorias de Falha**: contagem + % por `categoria_falha`
- **Peças Mais Trocadas**: count de itens com tipo='peca', agrupado por descricao.trim().toUpperCase()

## Histórico de sessões
### 2026-08-11
**Refatoração Frota/Manutenção + Análises expandidas (pedido do Salviano, via Claude Code na pasta local "Murici Control Tower" — repo real clonado nessa sessão pela primeira vez, ver seção Identificação):**
- **Frota**: adicionados `marca`, `ano`, `venc_licenciamento`, `venc_seguro` (com aviso visual se vencido/vence em ≤30d); `motorista_fixo_id` (já existia no schema desde o refactor de maio, nunca tinha UI) e `km_atual` agora editáveis no formulário; `tipo` deixou de ter CHECK restrito a VAN/VUC (curado só na UI agora); `status` manual restrito a ATIVO/INATIVO — MANUTENCAO só é setado automaticamente pelo fluxo de OS
- **Manutenção (OS)**: adicionados `motorista_id` (quem dirigia na quebra), `motorista_oficina_id` (quem levou à oficina — campo separado, adicionado numa 2ª rodada depois que o Salviano apontou que são papéis diferentes), `previsao_saida`, `categoria_falha`, `valor_total` (editável na abertura e no fechamento)
- **Schema**: FK `manutencao_os.placa → frota.placa` adicionada — fechou o gap que causou o incidente `FYE0A48` (23/05, ver mais abaixo); precisou backfillar `FYE0A48` em `frota` como `INATIVO` antes de conseguir criar a FK, já que a OS antiga referenciava uma placa nunca cadastrada
- **Análises**: expandida de 2 para 6 cards + gráfico de barras mensal (ver seção "Aba Manutenção — detalhes" acima)
- **Achado**: tabela legada `manutencao` (schema pré-refactor de maio, ~10 registros) segue órfã, não usada pelo app — decisão consciente de não mexer, fora de escopo
- **Verificação**: testado com Playwright (Chromium headless via `npx playwright`, instalado nessa sessão pois não havia browser automation disponível na máquina) contra o Supabase real, local e em produção — 0 erros de console nas duas rodadas
- Deploy: 2 commits (`3f4680c` schema+UI principal, `a438a6a` fix do campo de motorista duplo) + `vercel deploy --prod` via CLI (precisou `npx vercel` com login por device code — sessão não tinha credenciais salvas)

### 2026-07-30
**Retomada de desenvolvimento após ~2 meses parado (último commit era 23/05):**
- **Causa raiz de "parou":** projeto Supabase (`pneakxdlbyuysjorhjdu`) estava com status `INACTIVE` (pausado automaticamente por inatividade, plano free). Restaurado via MCP. Precisou antes pausar outro projeto (`transvale-v1`) pra liberar vaga — conta free só permite 2 projetos ativos simultâneos.
- **Achado de segurança — "demo" não é demo de verdade:** `sistema-gestao-transportadora.vercel.app` e `murici-ipiranga.vercel.app` são o **mesmo código do mesmo repositório GitHub**, sem nenhuma condição de hostname nas credenciais (`SURL`/`SKEY` hardcoded sem ternário, diferente do `_REAL` do Canais Críticos). O badge "MODO DEMO" (linha ~3235) é só cosmético. RLS confirmado aberto (`public`/`anon` FOR ALL) em praticamente todas as 15+ tabelas.
- **Correção de premissa:** a decisão de risco aceito de 28/07 (ver CLAUDE.md raiz) achava que não tinha CPF exposto — **errado**, a tabela `equipe` tem CPF real de 31 colaboradores, também sem proteção nenhuma. Reapresentado ao Salviano com o dado correto — **reafirmou conscientemente manter como está por enquanto** (30/07/2026), foco é retomar o desenvolvimento primeiro.
- **Limpeza (commit `0ca0153`):** removidos `fix_escala.js`/`replace_escala.js` (scripts de patch pontual de 13/05, já superados, usavam nomes de variável errados `SUPABASE_URL`/`SUPABASE_ANON_KEY`) e `.vercel/project.json.bak` (backup do link **errado**, apontava pro projeto demo antes da correção de 13/05 — mantido só confundiria). `CLAUDE.md` e `.vercel/project.json` versionados pela primeira vez.
- **Validado:** dados presentes no banco restaurado (frota=12, equipe=74, entrada_diaria=346, escala_rascunho=65, pagamentos_recarga=60, manutencao_os=11, etc.) — `recargas` e `processos_especificos` vazios são esperados (aba Recargas usa `entrada_diaria`, não a tabela `recargas`).
- **Não verificado ainda:** funcionamento visual real das 10 abas (sem navegador disponível na sessão) — recomendado o Salviano abrir o app e conferir.
- **Backlog restante:** revisitar separação demo/real de verdade (login + RLS `authenticated`, mesmo padrão já aplicado no Canais Críticos/Inbursa hoje) — adiado por decisão do usuário; bump de versões CDN (React 18.2.0/jsPDF 2.5.1/xlsx 0.18.5) — baixa prioridade, nenhuma está quebrada.

### 2026-05-12
- Implementada recarga manual (valor_recarga_manual_motorista/ajudante)

### 2026-05-13
- Aba Escala recriada do zero (código anterior perdido)
- Corrigido .vercel/project.json que apontava para projeto demo
- Adicionado export Excel na Escala com Mat. Ajudante 1 e 2
- Fix: nome da aba Excel não pode ter barra (fd retorna DD/MM/YYYY)

### 2026-05-23
- **RLS ativado (segurança Supabase):** Tabelas `pagamentos_recarga` e `manutencao_historico` estavam sem RLS. Supabase alertou por e-mail (17/05/2026). Aplicado via MCP: `ENABLE ROW LEVEL SECURITY` + política `anon_full_access` (FOR ALL TO anon USING (true) WITH CHECK (true)). Todas as 16 tabelas do projeto agora com `rls_enabled: true`. Nenhuma alteração de código necessária.
- **FYE0A48 — OS encerrada:** Placa não estava na tabela `frota` (nunca cadastrada). Tinha OS ABERTA em `manutencao_os` (id: 8842c0f3, CENTRAL TRUCK, entrada 23/04). Fechada: status=CONCLUIDO, data_retorno=2026-05-23, obs="Problemas no eixo - veiculo nao pertence mais a frota". Histórico em `entrada_diaria` (14/04 e 23/04) preservado intacto.
- **Fix SERVICO_INTERNO — Entrada do Dia:** `loadRefs` filtrava equipe só com `status:'ATIVO'` → colaboradores internos (ex: Thiago Ferreira Lima, mat.7710048426) não apareciam no select de Ajudante. Fix na linha 428: `{status:{op:'in',val:'(ATIVO,SERVICO_INTERNO)'}}`. Agora SERVICO_INTERNO aparece em Ajudante 1/2 mas não em Motorista (que filtra por funcao=MOTORISTA). Deploy em produção.

### 2026-05-16
- **Fix ranking — deduplicação de nomes:** função `nk()` normaliza chave do mapa com trim + NFD + remove acentos + maiúsculo + colapsa espaços internos — evita duplicatas por variação de digitação
- **Novo score Motorista (pesos):** Pontualidade 25% · Presença 25% · Recargas 20% · Saídas 15% · Volume 10% · Eficiência 5%
- **Novo score Ajudante (pesos):** Aparições na operação (entrada_diaria) 55% · Presença 25% · Recargas 15% · Volume 5% — sem pontualidade/eficiência (não capturados para ajudantes)
- **Coluna Presença** adicionada na tabela (barra azul, % dias trabalhados / total dias do período)
- **Colunas Tempo Médio, Cx/Hora e Pontualidade** ocultadas na view de Ajudantes
- **Legenda do score** atualizada e dinâmica por categoria
- **Correção de dados no Supabase (entrada_diaria + equipe):**
  - `JEFERSON GUIMARAES PAVARIN` → `JEFERSON GUIMARAES PAVARIM` (typo N→M)
  - `DIEGO CRUZ` / `DIEGO DA CRUZ BARBOSA` → `DIEGO CRUZ BARBOZA` (nome canônico)
  - `JOAO VICTOR QUITERIO DE SOUZA` → `JOÃO VICTOR QUITERIO DE SOUZA` (acento)
  - Espaço trailing no nome do Jeferson na tabela equipe também corrigido

# CLAUDE.md — Murici Ipiranga

## Identificação
- **App real:** https://murici-ipiranga.vercel.app
- **Código:** `C:\Users\Desktop\portfolio\murici-ipiranga\index.html`
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
| Frota | frota, manutencao | CRUD + manutenção |
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
- **manutencao_os**: id, placa, data_entrada, data_retorno, oficina, status (ABERTO/CONCLUIDO), km_entrada, km_retorno, observacoes, created_at
- **manutencao_itens**: id, os_id (FK → manutencao_os), placa, tipo (peca/servico), descricao, quantidade, data_troca, created_at

### Fluxo
1. "+ Novo Envio" → abre OS em manutencao_os + marca frota como MANUTENCAO
2. "📦 Itens / Peças" → modal para adicionar peças/serviços à OS (manutencao_itens)
3. "✓ Retornou" → fecha OS (status=CONCLUIDO) + marca frota como ATIVO

### Tabs da aba
- **Ativas**: cards com cor por criticidade (azul <10d, laranja 10-19d, amarelo 20-29d, vermelho 30d+) + KPIs
- **Histórico**: tabela expansível por linha — clica na linha para ver peças/serviços daquela OS
- **Análises**: ranking de veículos por frequência de manutenção + top 10 peças mais trocadas

### Análises — como funciona
- Carrega todos os registros de manutencao_os + manutencao_itens em memória
- Agrupamento feito em JS (sem GROUP BY no banco)
- Frequência: count de OS por placa + dias médios (só OS concluídas)
- Peças: count de itens com tipo='peca', agrupado por descricao.trim().toUpperCase()

## Histórico de sessões
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

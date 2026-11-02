# Plataforma de Gestão Operacional — Transportadora

Sistema operacional completo desenvolvido para empresa do segmento de transporte e distribuição. Centraliza o controle diário de entregas, frota, despesas e indicadores operacionais em uma única interface.

![Dashboard](https://img.shields.io/badge/status-demo-e8195a) ![Stack](https://img.shields.io/badge/stack-React%20%7C%20jsPDF%20%7C%20XLSX-06080f)

## Demonstração pública

Esta versão usa apenas dados fictícios gerados em memória. Não exige Supabase e não consulta ou altera dados da operação real. Cadastros, edições e exclusões são simulados na sessão e descartados ao recarregar a página. As bibliotecas de interface e exportação continuam sendo carregadas via CDN.

O ambiente operacional da Murici é uma implantação separada. Este código é destinado à demonstração pública do portfólio.

## Funcionalidades

- **Painel operacional** — visão do dia com volumes, receitas e indicadores em tempo real
- **Controle de frota** — cadastro de veículos, manutenções e disponibilidade
- **Gestão de entregas** — registro, acompanhamento e fechamento de rotas diárias
- **Despesas e reembolsos** — controle de custos operacionais por colaborador
- **Relatórios em PDF** — geração automática de relatórios diários e semanais
- **Export Excel** — planilhas de dados para análise posterior

## Stack

| Tecnologia | Uso |
|-----------|-----|
| React 18 | Interface (via CDN, sem build) |
| jsPDF + autoTable | Geração de relatórios PDF |
| SheetJS (XLSX) | Export de planilhas |
| JetBrains Mono | Tipografia monospace para dados |

## Como rodar

```bash
# Sem dependências — abrir direto no browser
open index.html

# Ou via servidor local
npx serve .
python -m http.server 8080
```

## Projeto

Desenvolvido para digitalizar o controle operacional de uma transportadora que gerenciava tudo em planilhas. A plataforma reduziu o tempo de fechamento diário e centralizou informações que antes ficavam dispersas entre diferentes colaboradores.

**Segmento:** Transporte e distribuição  
**Tipo:** Sistema operacional interno  
**Principais usuários:** Supervisores operacionais, motoristas e gestores

---

*Portfólio — [SalvianoLopes](https://github.com/SalvianoLopes)*

# Template — Relatório de Auditoria de Segurança

Use esta estrutura como guia ao montar o .docx (ver skill de docx para formatação).

---

## Capa
- Título: "Relatório de Auditoria de Segurança — [Nome do Sistema]"
- Data da análise
- Escopo (ex: "Código-fonte PHP, configuração de servidor e MySQL")

## 1. Sumário Executivo
- Tabela: contagem de achados por severidade (Crítica / Alta / Média / Baixa / Informativa)
- 3-5 frases descrevendo o risco geral do sistema e os pontos mais urgentes
- Se aplicável: recomendação de pentest profissional formal para sistemas de alto risco

## 2. Metodologia e Escopo
- O que foi analisado (arquivos, diretórios, se houve acesso ao servidor/banco)
- O que ficou fora do escopo (ex: "não houve acesso direto ao servidor MySQL; achados dessa camada são inferidos do código")
- Critério de severidade usado (referenciar as definições da skill)

## 3. Achados Detalhados

Organizar em três subseções, cada achado ordenado por severidade (Crítica primeiro):

### 3.1 Código PHP
Para cada achado:
- **[SEVERIDADE] Título curto do problema**
- Localização: `arquivo.php:linha`
- Descrição
- Trecho de código relevante (poucas linhas)
- Recomendação de correção

### 3.2 Servidor / Infraestrutura
(mesmo formato)

### 3.3 MySQL
(mesmo formato — indicar se é inspeção direta ou inferência)

## 4. Recomendações Priorizadas
- Lista ordenada: o que corrigir primeiro, segundo, etc. (geralmente segue a ordem de severidade, mas pode reagrupar por "quick wins" vs. mudanças estruturais)

## 5. Apêndice Técnico (opcional)
- Comandos usados para verificação (ex: greps, queries de inspeção do MySQL)
- Lista completa de arquivos analisados

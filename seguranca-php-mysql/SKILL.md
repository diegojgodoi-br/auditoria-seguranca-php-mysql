---
name: seguranca-php-mysql
description: Auditoria de segurança para aplicações PHP e bancos MySQL — analisa código-fonte PHP, configuração de servidor (php.ini, .htaccess, exposição de .env) e configuração/permissões do MySQL, e gera um relatório formal (.docx) com achados, severidade e recomendações. Use sempre que o usuário pedir para "analisar segurança", "auditar", "revisar vulnerabilidades", "fazer pentest de código", "checar SQL Injection/XSS/CSRF", verificar permissões de usuário do banco, ou revisar a estrutura/configuração de um sistema PHP+MySQL antes de reescrever, publicar ou colocar em produção. Também use quando o usuário mencionar um sistema legado em PHP que precisa de auditoria antes de manutenção.
license: MIT
metadata:
  autor: Diego Godoi (iacomousar.com.br)
  url: https://github.com/diegojgodoi-br/auditoria-seguranca-php-mysql
---

# Auditoria de Segurança — PHP + MySQL

Esta skill guia uma auditoria de segurança estruturada de aplicações PHP com backend MySQL, cobrindo três camadas: **código PHP**, **configuração de servidor/infraestrutura** e **configuração do MySQL**. O resultado final é um relatório formal em .docx, adequado para arquivar ou compartilhar com stakeholders.

Esta skill é para fins **defensivos** — auditar sistemas do próprio usuário ou sob sua responsabilidade, identificar vulnerabilidades e sugerir correções. Nunca gerar exploits funcionais, payloads de ataque prontos para uso, ou instruções que sirvam primariamente para atacar terceiros — o foco é sempre diagnóstico + remediação.

## Quando usar

- Usuário pede para analisar/auditar código PHP em busca de vulnerabilidades
- Usuário quer revisar a estrutura de um sistema antes de reescrever ou colocar em produção (ex: sistemas legados)
- Usuário quer checar configuração de servidor (php.ini, .htaccess, exposição de arquivos sensíveis)
- Usuário quer revisar permissões/configuração de usuários MySQL

## Fluxo de trabalho

### 1. Levantar escopo

Antes de começar, confirme com o usuário (se ainda não estiver claro):
- Caminho(s) do código-fonte a analisar (upload, ou pasta no filesystem)
- Se há acesso a arquivos de configuração (`php.ini`, `.htaccess`, `my.cnf`) ou só ao código da aplicação
- Se há acesso ao MySQL para consultar usuários/grants diretamente, ou só a scripts/dumps de configuração
- Ambiente: sistema já está em produção, ou é uma auditoria prévia a deploy/reescrita?

Se o usuário já colou o código ou indicou os arquivos, não precisa perguntar de novo — siga direto.

### 2. Analisar código PHP

Leia `references/php-code-vulnerabilities.md` para o checklist completo. Resumo das categorias:
- Injeção SQL (queries concatenadas, ausência de prepared statements/PDO)
- XSS (saída sem `htmlspecialchars`/escaping, uso de `innerHTML` refletindo input)
- CSRF (formulários/ações de estado sem token)
- Autenticação e sessão (hashing de senha, fixação/hijacking de sessão, cookies sem `HttpOnly`/`Secure`)
- Upload de arquivos (validação de tipo/extensão, path traversal, execução de arquivos enviados)
- Controle de acesso e permissões (IDOR, checagem de papel/role ausente)
- Inclusão de arquivos (LFI/RFI via `include`/`require` com input do usuário)
- Exposição de dados sensíveis (senhas/chaves hardcoded, mensagens de erro verbosas, `var_dump`/`print_r` esquecido)

Varra o código sistematicamente — não amostre apenas alguns arquivos se o usuário pediu auditoria completa. Use `grep`/busca por padrões perigosos (`$_GET`, `$_POST`, `mysqli_query`, `include`, `eval`, etc.) como ponto de partida, depois leia o contexto de cada ocorrência antes de classificar.

### 3. Analisar configuração de servidor/infraestrutura

Leia `references/server-infra-config.md`. Cobre:
- `php.ini`: `display_errors`, `expose_php`, `allow_url_include`, `register_globals`, limites de upload
- `.htaccess`: proteção de diretórios sensíveis, bloqueio de acesso a `.env`/`.git`/backups
- Exposição de arquivos: `.env`, `.git/`, arquivos `.sql` de backup, `phpinfo()` acessível publicamente
- Headers de segurança ausentes (CSP, X-Frame-Options, HSTS)

### 4. Analisar configuração do MySQL

Leia `references/mysql-security.md`. Cobre:
- Usuários com privilégios excessivos (`GRANT ALL`, uso de `root` pela aplicação)
- Usuários sem restrição de host (`'user'@'%'`)
- Senhas fracas ou ausentes, plugins de autenticação desatualizados
- Configurações em `my.cnf` (bind-address, `local-infile`, logging)
- Ausência de separação de privilégios entre ambientes (mesmo usuário para leitura e escrita/admin)

Se o usuário não tiver acesso direto ao servidor MySQL, baseie essa parte na análise da string de conexão e nos privilégios que o código efetivamente usa (ex: se o código só faz SELECT, mas conecta com um usuário que tem DROP/ALTER, já é um achado).

### 5. Consolidar achados e classificar severidade

Para cada achado, registre:
- **Categoria** (SQLi, XSS, config de servidor, etc.)
- **Localização** (arquivo:linha, ou nome do arquivo de config)
- **Severidade**: Crítica / Alta / Média / Baixa / Informativa
- **Descrição** do problema e por que é um risco
- **Recomendação** de correção concreta (não genérica — aponte a mudança específica no código/config)

Critério de severidade sugerido:
- **Crítica**: execução remota de código, SQLi em endpoint sem autenticação, credenciais expostas publicamente
- **Alta**: SQLi/XSS que exige autenticação prévia, upload sem validação, IDOR em dado sensível
- **Média**: CSRF sem token, headers de segurança ausentes, erro verboso exposto
- **Baixa**: boas práticas não seguidas mas sem exploração direta óbvia
- **Informativa**: observações de hardening, sem vulnerabilidade concreta

### 6. Gerar o relatório formal

O entregável padrão é um **documento Word (.docx)**. Se houver uma skill de docx disponível no ambiente, leia-a antes de criar o arquivo para seguir as boas práticas de formatação. Se o ambiente não conseguir gerar .docx (ou o usuário preferir), entregue o mesmo relatório em Markdown, seguindo a mesma estrutura.

Estrutura sugerida do relatório (ver `assets/estrutura-relatorio.md` para o template completo):
1. Capa / identificação (sistema auditado, data, autor da análise)
2. Sumário executivo (contagem de achados por severidade, principais riscos)
3. Metodologia e escopo
4. Achados detalhados, agrupados por camada (Código PHP / Servidor / MySQL), ordenados por severidade
5. Recomendações priorizadas (o que corrigir primeiro)
6. Apêndice técnico (se aplicável: trechos de código problemáticos, comandos de verificação)

Depois de gerar o relatório, entregue o arquivo ao usuário e resuma em poucas linhas os achados críticos e altos.

## Notas importantes

- Sempre priorize achados por severidade — não enterre um problema crítico no meio de dez itens informativos.
- Ao citar código problemático, inclua o trecho relevante (poucas linhas) só o suficiente para ilustrar o problema — não é necessário reproduzir o arquivo inteiro.
- Se o volume de código for muito grande para revisar tudo em uma sessão, comunique isso ao usuário e proponha priorizar por: pontos de entrada de input do usuário, endpoints de autenticação, e arquivos que manipulam upload/pagamento/dados sensíveis primeiro.
- Esta skill não substitui uma auditoria de segurança profissional formal (pentest com escopo contratado) para sistemas críticos — mencione isso no relatório se o sistema auditado parecer ser de alto risco (ex: processa pagamentos, dados de saúde).

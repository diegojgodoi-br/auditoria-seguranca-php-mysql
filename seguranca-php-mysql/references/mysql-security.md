# Checklist — Segurança MySQL

## Usuários e privilégios

Se houver acesso para consultar diretamente (`SELECT user, host FROM mysql.user;` e `SHOW GRANTS FOR 'user'@'host';`):

- **Privilégio excessivo**: a aplicação conecta com usuário que tem `GRANT ALL PRIVILEGES` quando só precisa de `SELECT, INSERT, UPDATE, DELETE` nas tabelas específicas do sistema
- **Uso de root pela aplicação**: string de conexão da aplicação usa `root` — crítico, qualquer SQLi vira controle total do banco
- **Host wildcard**: usuário definido como `'app_user'@'%'` em vez de restrito ao IP/host do servidor de aplicação — amplia superfície de ataque se a credencial vazar
- **Falta de segregação por ambiente**: mesmo usuário/senha usado em dev, homologação e produção
- **Senha fraca ou ausente**: usuários com `authentication_string` vazio, ou senha idêntica ao nome do usuário/padrão de fábrica
- **Múltiplas responsabilidades no mesmo usuário**: um único usuário faz leitura de relatórios E escrita transacional E tarefas administrativas — dificulta auditoria e amplia dano em caso de comprometimento

Se **não** houver acesso direto ao servidor MySQL, infira a partir do código:
- A string de conexão usa qual usuário? Esse usuário aparenta ter mais privilégio do que as queries do código realmente usam?
- Há queries `DROP`, `ALTER`, `GRANT` no código da aplicação (fora de scripts de migração controlados)? Isso sugere privilégio desnecessário disponível em tempo de execução.

## Configuração (my.cnf / my.ini)

Diretivas a verificar quando o arquivo estiver disponível:

| Diretiva | Recomendado | Risco |
|---|---|---|
| `bind-address` | IP específico, não `0.0.0.0` se não precisar de acesso externo | Exposição do MySQL na rede/internet |
| `local-infile` | `0` (desabilitado) salvo necessidade explícita | `LOAD DATA LOCAL INFILE` pode ser abusado via SQLi para leitura de arquivos |
| `skip-networking` | Considerar `ON` se app e banco estão no mesmo host | Reduz superfície de rede |
| `secure-file-priv` | Restrito a diretório específico | Controla onde `LOAD DATA`/`SELECT INTO OUTFILE` podem ler/escrever |
| `log_error` / `general_log` | Configurado, mas sem logar dados sensíveis em texto puro | Auditoria vs. vazamento de dados em logs |

## Estrutura e dados

- Colunas de senha: devem armazenar hash (bcrypt/argon2 via `password_hash` do PHP), nunca texto puro ou hash reversível
- Dados sensíveis (CPF, cartão, etc.) sem criptografia em repouso quando aplicável ao contexto do negócio
- Backups de banco (`.sql`) acessíveis na webroot (ver também `server-infra-config.md`)

## Ao registrar achados desta camada

Sempre diferencie no relatório se o achado veio de:
1. **Inspeção direta do MySQL** (mais confiável — cite a query usada)
2. **Inferência a partir do código da aplicação** (deixe claro que é inferência, e recomende validação direta no servidor como próximo passo)

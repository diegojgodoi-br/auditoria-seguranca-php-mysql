# Checklist — Vulnerabilidades em Código PHP

Para cada item, procure o padrão no código, avalie o contexto (o input chega de onde? é sanitizado antes?) e registre como achado se aplicável.

## 1. SQL Injection

Padrões de risco:
- Concatenação de string diretamente em query: `"SELECT * FROM x WHERE id = " . $_GET['id']`
- Uso de `mysqli_query`/`mysql_query` (extensão antiga, já removida do PHP 7+) sem prepared statements
- `PDO` usado sem binding de parâmetros (`->query()` com string montada em vez de `->prepare()` + `->execute([...])`)
- Interpolação direta em query dentro de f-string/heredoc: `"...WHERE nome = '$nome'"`

Correção esperada: prepared statements com parâmetros bindados (`PDO::prepare` + `execute`, ou `mysqli_prepare` + `bind_param`). Nunca sanitização manual via `addslashes`/`mysqli_real_escape_string` como única defesa — é insuficiente em vários contextos (ex: LIKE, ORDER BY, nomes de coluna).

## 2. Cross-Site Scripting (XSS)

Padrões de risco:
- Output de `$_GET`/`$_POST`/dados do banco direto no HTML sem `htmlspecialchars()`
- Uso de `echo`/`print` de dados vindos de formulário em atributos HTML sem escaping
- JavaScript inline que injeta valores PHP sem `json_encode` (para contexto JS) ou escaping adequado
- Campos "ricos" (ex: comentários, descrições) sem sanitização de HTML permitido (ausência de allowlist tipo HTMLPurifier)

Correção esperada: `htmlspecialchars($valor, ENT_QUOTES, 'UTF-8')` em toda saída de dados não confiáveis; `json_encode` para inserir dados PHP em contexto JavaScript.

## 3. CSRF (Cross-Site Request Forgery)

Padrões de risco:
- Formulários que alteram estado (criar/editar/excluir) sem token CSRF
- Ações destrutivas via `GET` (ex: `?action=delete&id=5` sem confirmação nem token)
- Ausência de verificação de `Origin`/`Referer` em endpoints sensíveis

Correção esperada: token CSRF único por sessão/formulário, validado no servidor antes de processar a ação; usar métodos HTTP corretos (mutações via POST/DELETE, não GET).

## 4. Autenticação e Sessão

Padrões de risco:
- Senhas armazenadas em texto puro, com `md5()`/`sha1()` sem salt, ou hash fraco
- Ausência de `password_hash()`/`password_verify()` (funções nativas do PHP recomendadas desde 5.5)
- Session ID não regenerado após login (`session_regenerate_id()` ausente) → session fixation
- Cookies de sessão sem flags `HttpOnly`, `Secure`, `SameSite`
- Ausência de rate limiting / bloqueio em tentativas de login (força bruta)
- Comparação de tokens/senhas com `==` em vez de `hash_equals()` (timing attack)

Correção esperada: `password_hash()`/`password_verify()`, `session_regenerate_id(true)` pós-login, cookies com flags de segurança configuradas via `session_set_cookie_params()`.

## 5. Upload de Arquivos

Padrões de risco:
- Validação de tipo de arquivo só pela extensão ou `$_FILES['x']['type']` (facilmente falsificável pelo cliente)
- Ausência de checagem do conteúdo real do arquivo (magic bytes / `finfo`)
- Arquivos salvos com o nome original do usuário (path traversal via `../../`)
- Diretório de upload dentro da webroot e executável (permite upload de `.php` disfarçado e execução remota)

Correção esperada: validar tipo real via `finfo_file`, gerar nome de arquivo novo (não confiar no nome enviado), salvar fora da webroot ou em diretório sem permissão de execução, allowlist de extensões.

## 6. Controle de Acesso / IDOR

Padrões de risco:
- Endpoints que aceitam um ID de recurso (`?pedido_id=123`) sem checar se o usuário logado tem permissão sobre aquele recurso
- Ausência de checagem de papel/role em rotas administrativas (ex: só checa se está logado, não se é admin)
- Lógica de autorização duplicada e inconsistente entre páginas

Correção esperada: checagem centralizada de propriedade do recurso (o registro pertence ao usuário logado?) e de papel, idealmente em uma camada única reutilizada por todas as rotas.

## 7. Inclusão de Arquivos (LFI/RFI)

Padrões de risco:
- `include`/`require`/`include_once` com caminho vindo de `$_GET`/`$_POST` sem allowlist
- `allow_url_include` habilitado (permite RFI incluindo URL remota)

Correção esperada: nunca usar input do usuário diretamente no caminho de include; usar allowlist fixa de páginas/templates válidos.

## 8. Exposição de Dados Sensíveis

Padrões de risco:
- Credenciais de banco, chaves de API ou tokens hardcoded no código-fonte
- `display_errors` ligado em produção, expondo stack trace com caminhos do servidor
- `var_dump()`, `print_r()`, `error_log()` de dados sensíveis esquecidos em código de produção
- Mensagens de erro de login que revelam se o usuário existe ("senha incorreta" vs "usuário não encontrado")

Correção esperada: credenciais em variáveis de ambiente/arquivo `.env` fora da webroot, `display_errors = Off` em produção com log de erros server-side, mensagens de erro genéricas para o usuário final.

## Como buscar candidatos rapidamente

Pontos de partida úteis para grep (depois leia o contexto de cada resultado — não classifique só pelo match):

```
grep -rn "\$_GET\|\$_POST\|\$_REQUEST" --include="*.php" .
grep -rn "mysqli_query\|->query(" --include="*.php" .
grep -rn "include \$\|require \$\|include_once \$" --include="*.php" .
grep -rn "eval(\|system(\|exec(\|shell_exec(" --include="*.php" .
grep -rn "md5(\|sha1(" --include="*.php" .
grep -rn "move_uploaded_file" --include="*.php" .
```

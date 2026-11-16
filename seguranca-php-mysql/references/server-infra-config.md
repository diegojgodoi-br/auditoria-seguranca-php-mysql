# Checklist — Configuração de Servidor / Infraestrutura

## php.ini

Diretivas a verificar:

| Diretiva | Valor recomendado (produção) | Risco se mal configurado |
|---|---|---|
| `display_errors` | `Off` | Expõe stack trace, caminhos internos, versões |
| `expose_php` | `Off` | Vaza versão do PHP no header `X-Powered-By` |
| `allow_url_include` | `Off` | Permite RFI (inclusão remota de arquivo) |
| `allow_url_fopen` | `Off` (se não necessário) | Reduz superfície de SSRF/RFI |
| `register_globals` | N/A (removido do PHP 5.4+) | Se ainda existir em legado, é crítico |
| `session.cookie_httponly` | `On` | Sem isso, cookie de sessão acessível via JS (XSS→session hijack) |
| `session.cookie_secure` | `On` (se HTTPS) | Sem isso, cookie pode vazar em conexão não criptografada |
| `upload_max_filesize` / `post_max_size` | Ajustado ao mínimo necessário | Limites excessivos facilitam DoS |
| `disable_functions` | Deve incluir `exec,system,shell_exec,passthru` se não usados | Reduz impacto de RCE caso ocorra |

## .htaccess (Apache) / equivalente Nginx

Verificar:
- Bloqueio de acesso direto a `.env`, `.git/`, `.sql`, `.log`, arquivos de config (`config.php` se guardar segredo em texto)
- `Options -Indexes` (evita listagem de diretório)
- Bloqueio de acesso a pastas de upload quanto à execução de PHP (impedir que um `.php` enviado seja executado)
- Redirecionamento forçado para HTTPS

Exemplo de regra ausente comum (Apache):
```apache
<FilesMatch "^\.env|\.git|\.sql$">
    Require all denied
</FilesMatch>
```

## Exposição de arquivos sensíveis

Checar se são acessíveis publicamente via HTTP:
- `.env` (credenciais de banco, chaves de API)
- `.git/` (histórico completo do repositório, possivelmente com segredos antigos)
- Backups (`backup.sql`, `dump.sql`, `*.bak`, `*.zip` na webroot)
- `phpinfo()` exposto em algum arquivo esquecido (`info.php`, `test.php`)
- Diretórios `/admin`, `/config`, `/logs` sem autenticação nem bloqueio de listagem

## Headers de segurança HTTP ausentes

Verificar se a aplicação envia (via código ou config de servidor):
- `Content-Security-Policy` — mitiga XSS
- `X-Frame-Options: DENY/SAMEORIGIN` — mitiga clickjacking
- `X-Content-Type-Options: nosniff`
- `Strict-Transport-Security` — força HTTPS
- `Referrer-Policy`

Ausência desses headers geralmente é severidade **Média**, não crítica isoladamente — mas soma-se a outros achados.

# Anubis devpod

Devcontainer Linux (Debian Bookworm) amb Go per executar dues eines d'[Anubis](https://github.com/TecharoHQ/anubis):

- [`iplist2rule`](https://anubis.techaro.lol/docs/admin/iplist2rule) converteix una llista de bloqueig d'IPs en un fitxer de política.
- [`robots2policy`](https://anubis.techaro.lol/docs/admin/robots2policy/) converteix un `robots.txt` en regles de política amb expressions CEL.

La imatge parteix de `golang:1.26.9-bookworm`. Anubis requereix Go 1.26.3 o superior. Les dues eines s'instal·len en construir la imatge i queden a `/go/bin`, que ja és al `PATH`.

## Obrir el contenidor

A Cursor, obre la paleta d'ordres i tria **Dev Containers: Reopen in Container**.

Amb [DevPod](https://devpod.sh/):

```bash
devpod up .
```

## iplist2rule

Conversió bàsica des d'una URL. L'acció per defecte és `DENY`:

```bash
iplist2rule https://raw.githubusercontent.com/7c/torfilter/refs/heads/main/lists/txt/torfilter-1m-flat.txt filter-tor.yaml
```

Permetre totes les IPs de la llista:

```bash
iplist2rule --action ALLOW https://raw.githubusercontent.com/7c/torfilter/refs/heads/main/lists/txt/torfilter-1m-flat.txt filter-tor.yaml
```

Afegir pes a les peticions que coincideixen:

```bash
iplist2rule --action WEIGH --weight 20 https://raw.githubusercontent.com/7c/torfilter/refs/heads/main/lists/txt/torfilter-1m-flat.txt filter-tor.yaml
```

| Flag | Descripció | Per defecte |
| --- | --- | --- |
| `--action` | Acció d'Anubis: `ALLOW`, `DENY` o `WEIGH` | `DENY` |
| `--rule-name` | Nom de la regla, en kebab-case | inferit del nom del fitxer |
| `--weight` | Punts de pes quan `--action=WEIGH` | `0` |

Importa el fitxer generat des de la política principal:

```yaml
bots:
  - import: "./filter-tor.yaml"
```

Per actualitzar l'eina a l'última versió publicada:

```bash
go install github.com/TecharoHQ/anubis/utils/cmd/iplist2rule@latest
```

## robots2policy

Conversió bàsica des d'una URL. El resultat surt per la sortida estàndard i l'acció per defecte dels camins prohibits és `CHALLENGE`:

```bash
robots2policy -input https://www.example.com/robots.txt
```

Convertir un fitxer local a YAML:

```bash
robots2policy -input robots.txt -output policy.yaml
```

Convertir amb acció i format propis:

```bash
robots2policy -input robots.txt -action DENY -format json
```

| Flag | Descripció | Per defecte |
| --- | --- | --- |
| `-input` | Ruta o URL del `robots.txt` (`-` per a stdin) | obligatori |
| `-output` | Fitxer de sortida (`-` per a stdout) | stdout |
| `-format` | Format de sortida: `yaml` o `json` | `yaml` |
| `-action` | Acció per als camins prohibits: `ALLOW`, `DENY`, `CHALLENGE` o `WEIGH` | `CHALLENGE` |
| `-name` | Prefix del nom de la política | `robots-txt-policy` |
| `-crawl-delay-weight` | Ajust de pes per a les regles de crawl-delay | `3` |
| `-deny-user-agents` | Acció per als user-agents de la llista negra | `DENY` |

Importa el fitxer generat des de la política principal:

```yaml
bots:
  - import: "./robots-policy.yaml"
```

Per actualitzar l'eina a l'última versió publicada:

```bash
go install github.com/TecharoHQ/anubis/cmd/robots2policy@latest
```

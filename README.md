# Anubis devpod

Devcontainer Linux (Debian Bookworm) amb Go per executar [`iplist2rule`](https://anubis.techaro.lol/docs/admin/iplist2rule), l'eina d'[Anubis](https://github.com/TecharoHQ/anubis) que converteix una llista de bloqueig d'IPs en un fitxer de política.

La imatge parteix de `golang:1.26.9-bookworm`. Anubis requereix Go 1.26.3 o superior. `iplist2rule` s'instal·la en construir la imatge i queda a `/go/bin`, que ja és al `PATH`.

## Obrir el contenidor

A Cursor, obre la paleta d'ordres i tria **Dev Containers: Reopen in Container**.

Amb [DevPod](https://devpod.sh/):

```bash
devpod up .
```

## Ús

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

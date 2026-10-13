# Versions d'Anubis i polítiques

Referència per no aplicar una política d'una versió a un binari que encara no la coneix. Revisat contra el tag **v1.28.1** (9 d'octubre de 2026). L'estable anterior immediat és v1.28.0 (8 d'octubre de 2026).

La web `anubis.techaro.lol` està darrere del propi Anubis i sovint respon amb el challenge. El codi del tag és la font fiable: [github.com/TecharoHQ/anubis](https://github.com/TecharoHQ/anubis).

## Veure els fitxers `(data)/`

Un `import: (data)/...` no es descarrega en arrencar. És la carpeta `data/` incrustada dins del binari en el moment de compilar-lo. Es resol contra la versió que corre, no contra `main`.

Per llegir-los, obre el tag que coincideix amb el servidor. Exemple per a la v1.28.1: [data/](https://github.com/TecharoHQ/anubis/tree/v1.28.1/data).

Per veure la còpia exacta del binari que corre, al servidor:

```sh
anubis --version
anubis --extract-resources ./var/anubis-resources
```

Això deixa `./var/anubis-resources/data/`. En un paquet natiu també hi és a `/usr/share/docs/anubis/data`. El flag `--extract-resources` existeix des de la v1.16.0.

El devcontainer d'aquest repo només instal·la `iplist2rule` i `robots2policy`. Per extreure la còpia localment, cal el binari del mateix tag que producció:

```sh
go install github.com/TecharoHQ/anubis/cmd/anubis@v1.28.1
anubis --extract-resources ./var/anubis-resources
```

`(data)/common/keep-internet-working.yaml` a la v1.28.1 permet `/.well-known/`, `favicon.ico` (i png, gif, jpg, jpeg, svg), `robots.txt` i `sitemap.xml`.

## Què pots copiar segons la versió

| Des de | El que pots fer servir |
| --- | --- |
| v1.17.0 | `import: (data)/...` |
| v1.18.0 | Expressions CEL: `headers`, `host`, `method`, `path`, `query`, `remoteAddress`, `userAgent` |
| v1.20.0 | `WEIGH`, llindars (`thresholds`), GeoIP i ASN (llavors via Thoth) |
| v1.21.0 | `load_1m`, `load_5m`, `load_15m` |
| v1.21.1 | `missingHeader` |
| v1.22.0 | `segments`. El solver `slow` arrenca amb un avís |
| v1.23.0 | `contentLength`. Diverses comprovacions alhora passen a ser AND. Import de `(data)/meta/default-config.yaml` |
| v1.24.0 | DNS a CEL: `verifyFCrDNS`, `reverseDNS`, `lookupHost`, `arpaReverseIP`, `regexSafe`. `report_as` obsolet |
| v1.27.0 | Imports amb comodins, per exemple `(data)/crawlers/*.yaml` |
| v1.28.0 | GeoIP local (MaxMind) en lloc de Thoth. Llistes d'IP dinàmiques. Challenges WebAssembly, apagats per defecte |

Si copies una política de `main` a una versió més antiga, Anubis pot no arrencar: el fitxer `(data)/` no existeix, o una clau encara no es reconeix.

## Canvis que trenquen un fragment copiat de la documentació

El `botPolicies.yaml` que circula en exemples antics encara diu que GeoIP i ASN necessiten una subscripció a Thoth. A la **v1.28.0** Thoth s'ha tret. `THOTH_URL`, `THOTH_TOKEN` i `THOTH_INSECURE` queden obsolets i només avisen a l'arrencada.

A la v1.28.1 cal una base MaxMind local. Al final de `botPolicies.yaml` està comentada:

```yaml
# geoip:
#   asn:
#     path: /var/lib/anubis/GeoLite2-ASN.mmdb
#   country:
#     path: /var/lib/anubis/GeoLite2-Country.mmdb
#   auto_update:
#     account_id: 123456
#     license_key_env: MAXMIND_LICENSE_KEY
#     interval: 24h
```

Les regles `countries-with-aggressive-scrapers` (BR, CN) i `aggressive-asns-without-functional-abuse-contact` (AS13335 Cloudflare, AS136907 Huawei Cloud, AS45102 Alibaba Cloud) continuen al fitxer per defecte, amb `WEIGH` i `adjust: 10`. Sense la base GeoIP no fan res.

`report_as` (mentir sobre la dificultat del challenge) es va marcar obsolet a la **v1.24.0**. A la v1.28.1 ja no surt al comentari de `generic-bot-catchall`. L'algorisme `slow` es va retirar a la **v1.22.0**: si el deixes, el procés arrenca i escriu un avís.

A la **v1.23.0** diverses comprovacions a la mateixa regla (per exemple `user_agent_regex` i `remote_addresses` junts) passen de OR a AND: la regla només coincideix si es compleixen totes.

## Càrrega del sistema

`load_1m`, `load_5m` i `load_15m` existeixen des de la **v1.21.0**. A la v1.28.1 les dues regles segueixen comentades al fitxer per defecte:

```yaml
# - name: high-load-average
#   action: WEIGH
#   expression: load_1m >= 10.0 # el nombre ha d'acabar en .0
#   weight:
#     adjust: 20

# - name: low-load-average
#   action: WEIGH
#   expression: load_15m <= 4.0
#   weight:
#     adjust: -10
```

`WEIGH` no desafia per si sol. Suma o resta punts, i els llindars decideixen. Els llindars per defecte de la v1.28.1:

| Pes | Llindar | Què passa |
| --- | --- | --- |
| `<= 0` | `minimal-suspicion` | `ALLOW` |
| `1–9` | `mild-suspicion` | `metarefresh`, dificultat 1 |
| `10–19` | `moderate-suspicion` | proof-of-work `fast`, dificultat 2 |
| `20–29` | `mild-proof-of-work` | dificultat 4 |
| `>= 30` | `extreme-suspicion` | dificultat 6 |

La regla genèrica de navegador (`Mozilla|Opera`) ja suma 10. Amb `load_1m >= 10.0` i `adjust: 20`, un navegador normal passa de dificultat 2 a 4 mentre la màquina va carregada.

La càrrega no sap quants nuclis té la màquina. El `10.0` de l'exemple és alt en un equip de 4 nuclis i baix en un de 16. La guia recomana el doble de nuclis per a càrrega alta i la meitat per a càrrega baixa. La comparació ha d'acabar en `.0` perquè CEL ho tracti com a decimal.

La càrrega és la del sistema operatiu on corre Anubis. Si el backend és una altra màquina, `load_1m` no veu ni la CPU ni l'espera de disc o de base de dades d'aquell servei. La regla de càrrega baixa (`adjust: -10`) cancel·la els 10 punts del navegador i el deixa passar sense challenge. Només té sentit si Anubis i el backend comparteixen el mateix sistema.

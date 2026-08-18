# Warum dieser Fork existiert

Fork von [jezweb/twenty-mcp](https://github.com/jezweb/twenty-mcp) (MIT), angelegt am 2026-08-18.

## Grund

Fünf Write-Mutationen deklarierten `$id` als `ID!` bzw. `String!`, während Twentys GraphQL-Schema durchgängig `UUID!` erwartet. Jeder betroffene Call brach ab mit:

```
Variable "$id" of type "ID!" used in position expecting type "UUID!"
```

Betroffen: `update_contact`, `update_company`, `update_opportunity`, `link_opportunity_to_company`, `transfer_contact_to_company`. Creates und Reads liefen immer.

Upstream ist der Bug seit Februar 2026 als [Issue #34](https://github.com/jezweb/twenty-mcp/issues/34) gemeldet — ohne Maintainer-Antwort. Der identische Fix lag als [PR #36](https://github.com/jezweb/twenty-mcp/pull/36) vor und wurde kommentarlos ungemergt geschlossen. Letzter Maintainer-Push: Januar 2026, fünf offene PRs. Deshalb Fork statt PR.

## Was geändert ist

Ein Commit, fünf Zeilen: `src/client/twenty-client.ts`, `ID!`/`String!` → `UUID!`. Verifiziert per Schema-Introspection gegen `twenty.noord.co` und per Smoke-Test über den echten MCP-stdio-Pfad.

Branch `fix/graphql-uuid-types` hält den Fix isoliert, falls er doch mal als PR rausgeht.

## Wie er eingebunden ist

`.mcp.json` in *Dans Agent Office* zeigt direkt auf `dist/index.js` dieses Checkouts — nicht mehr auf das global installierte npm-Paket. `dist/` ist gitignored, also nach jedem Pull neu bauen:

```bash
npm install --ignore-scripts && npm run build
```

Danach **Claude Code neu starten**, sonst läuft der alte Code weiter im Speicher.

## Bekannte, NICHT gefixte Defekte

[Issue #35](https://github.com/jezweb/twenty-mcp/issues/35) dokumentiert ein Audit vom Februar 2026: 20 von 29 Tools kaputt. Nicht angefasst sind unter anderem `get_tasks`, `create_task`, `create_note`, `get_activities`, `filter_activities` (Twenty 2.x hat `body` zu `bodyV2` umbenannt), `create_comment` (Mutation entfernt) und `search_opportunities` (ungültiges `skip`-Argument). Für diese Operationen die Twenty-REST-API direkt nutzen.

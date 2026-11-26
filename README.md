# FlareSolverr

Gets HolyArrs Sources past a site's bot check (Cloudflare, DDoS-Guard) with the
[FlareSolverr](https://github.com/FlareSolverr/FlareSolverr) the owner runs.
Needs HolyArrs 1.3.2 or newer.

How to set it up in HolyArrs:
[Get past bot checks with FlareSolverr](https://github.com/HolyArrs/HolyArrs/blob/main/docs/downloads/flaresolverr.md).

## What it does

When a plain feed call meets a bot check, HolyArrs hands that one page to this
plugin through the `botcheck` hook (permission `sources.botcheck`). The plugin:

1. long-polls `GET /api/v1/plugin-hooks/botcheck` on `HOLYARRS_PLUGIN_TARGET` with
   `HOLYARRS_PLUGIN_TOKEN`;
2. sends FlareSolverr `POST <address>/v1` with
   `{"cmd": "request.get", "url": ..., "maxTimeout": 60000, "proxy": ...}`, the
   Source's proxy passed on when it has one;
3. answers `POST /api/v1/plugin-hooks/botcheck` with the page FlareSolverr ended
   on, its status, cookies and User-Agent, or with `error` and a plain reason.

HolyArrs keeps all the rules: which pages may be handed over, that the page stays
on the Source's site and address rule, and which cookies it keeps.

It answers with an error when the address is not set, FlareSolverr does not
answer, takes over 68 seconds, answers its own error, ends on the check page
still, or sends a page over 16 MB. After FlareSolverr does not answer, it leaves
it alone for 10 seconds, doubling up to 2 minutes. When HolyArrs does not answer
the poll, it waits 1 second, doubling up to a minute. It opens up to 3 pages at
once, and never logs a page's address, its key, the proxy or a cookie.

## Setting

| Key | Arrives as | Default |
|---|---|---|
| `address` | `HOLYARRS_PLUGIN_SETTING_ADDRESS` | `http://flaresolverr:8191` |

The plugin lists no websites. The owner adds FlareSolverr's name and port, such
as `flaresolverr:8191`, under **Also allow your own server** on the Plugins page.

## Build and test

```sh
cd flaresolverr
go vet ./...
go test -race ./...
go build -trimpath .
```

The tests run against a fake HolyArrs hook and a fake FlareSolverr.

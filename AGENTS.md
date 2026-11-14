# Repository notes

## What this is

`nobitex-mcp` is a Python MCP server exposing the Nobitex exchange API (mainnet + testnet).

## Layout

- `src/nobitex_mcp/client.py` — HTTP client, Token + Ed25519 auth, signing, errors, read-only guard
- `src/nobitex_mcp/server.py` — all MCP tools (78) and entry point
- `src/nobitex_mcp/config.py` — env/.env settings
- `tests/` — unit tests + opt-in live testnet tests

## Commands

```bash
uv sync --extra dev
uv run pytest                       # unit tests
NOBITEX_LIVE_TEST=1 uv run pytest   # + live testnet public endpoints
uv run nobitex-mcp                  # stdio server
```

## Nobitex specifics (verified against apidocs.nobitex.ir)

- Base URLs: `https://apiv2.nobitex.ir` (mainnet), `https://testnetapiv2.nobitex.ir` (testnet)
- Websocket: `wss://ws.nobitex.ir` / `wss://ws.testnet.nobitex.ir`
- Auth: `Authorization: Token <token>` OR Ed25519 headers
  `Nobitex-Key`, `Nobitex-Signature` (URL-safe base64), `Nobitex-Timestamp` (unix seconds)
- Signature message: `timestamp + METHOD + full_path + raw_body`
  where `full_path` includes the query string and `raw_body` is the exact JSON bytes sent.
- `User-Agent` should be `TraderBot/<name-and-version>`.
- Some read operations are POST (`/users/wallets/list`, `/market/orders/status`, portfolio, etc.).
- OHLC (`/market/udf/history`) uses UDF keys (`s`, `t`, `o`, `h`, `l`, `c`, `v`), not `status`.
- Requests must originate from an Iranian IP; testnet is reachable internationally.

---
description: Every host, IP and port, and how to pick one
icon: earth-europe
---

# Regions and endpoints

## Iris endpoints

| Region | Host | IPv4 | QUIC | QUIC, MEV protected | Ghost QUIC | WebSocket |
| --- | --- | --- | --- | --- | --- | --- |
| Global edge | `edge.astralane.io` | | | | | |
| Frankfurt | `fr.gateway.astralane.io` | `185.191.117.97` | 7000 | 9000 | 6000 | 5500 |
| Frankfurt 2 | `fr2.gateway.astralane.io` | `45.139.132.160` | 7000 | 9000 | 6000 | 5500 |
| Amsterdam | `ams.gateway.astralane.io` | `64.130.43.43` | 7000 | 9000 | 6000 | 5500 |
| Amsterdam 2 | `ams2.gateway.astralane.io` | `84.32.186.73` | 7000 | 9000 | 6000 | 5500 |
| London | `lon.gateway.astralane.io` | `63.254.168.71` | 7000 | 9000 | 6000 | 5500 |
| Limburg | `lim.gateway.astralane.io` | `162.19.222.232` | 7000 | 9000 | 6000 | 5500 |
| Lithuania | `lit.gateway.astralane.io` | `84.32.97.47` | 7000 | 9000 | 6000 | 5500 |
| New York | `ny.gateway.astralane.io` | `64.130.45.19` | 7000 | 9000 | 6000 | 5500 |
| Los Angeles | `la.gateway.astralane.io` | `74.118.142.151` | 7000 | 9000 | 6000 | 5500 |
| Tokyo | `jp.gateway.astralane.io` | `189.1.164.31` | 7000 | 9000 | 6000 | 5500 |
| Singapore | `sg.gateway.astralane.io` | `67.209.54.176` | 7000 | 9000 | 6000 | 5500 |

Paths on every host: `/iris` (JSON-RPC), `/iris2` (plain text), `/irisb` (binary), `/gethealth` (health check). WebSocket paths are `/iris` and `/mev-protect`. Every region has the same ports and paths.

## Which region to use

1. **Start with the one nearest your servers.** Most stake sits in Europe. In Astralane's August analysis, Frankfurt held 29.6% of Solana stake and Amsterdam 18.2%.
2. **Time two or three hosts from your own server** and keep the fastest:

```bash
for h in fr ams lon ny; do
  printf "%s " $h; curl -s -o /dev/null -w "%{time_total}\n" https://$h.gateway.astralane.io/gethealth
done
```

3. **Also send to the global edge.** Sending each transaction to your nearest region and to `edge.astralane.io` covers leaders on other continents.
4. **Use the IP for QUIC.** Connecting by IP skips DNS. The regional hostnames also work.

## Shred and postpack relays

| Service | Regions | Address |
| --- | --- | --- |
| Shreds (UDP) | Frankfurt, New York | Set up in the portal |
| Decoded shreds (gRPC) | Frankfurt, New York | [Decoded shreds](../data-streams/decoded-shreds/README.md) |
| Postpacks (gRPC) | Frankfurt, Amsterdam, New York, Singapore, Tokyo | [Postpacks](../data-streams/postpacks.md) |

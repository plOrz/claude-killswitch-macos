# Changelog

## 1.0.2 - 2026-11-09

- Add Anthropic's newly published inbound IPv4 range to the physical-interface block table.
- Reinstalling this release refreshes the managed `pf` anchor with both current inbound IPv4 ranges.

## 1.0.1 - 2026-10-31

- Use the absolute installed path in post-install commands and guidance.
- Preserve the executable bit for the main script in Git.
- Remove an unused IPv6 constant; public IPv6 and NAT64 blocking is unchanged.

## 1.0.0 - 2026-10-31

- Initial public release; public versioning starts at 1.0.0.
- Physical-interface blocking for Anthropic IPv4.
- Public IPv6 and NAT64 blocking while preserving local IPv6 scopes.
- Boot-time root LaunchDaemon with runtime self-healing.
- Reversible `pf.conf` integration and state-aware uninstall.
- Non-Anthropic A/B verification flow.

# Security policy

## Reporting a vulnerability

Please open a GitHub issue without including credentials, tokens, private IP addresses, proxy configuration, or account information.

For a report that would expose sensitive details, first open a minimal issue asking for a private reporting channel.

## Security assumptions

- The installer and daemon run as root because macOS `pf` requires administrative privileges.
- The installed executable and LaunchDaemon plist are root-owned and not writable by ordinary users.
- The project validates candidate `pf` syntax before replacing the active configuration.
- Failed verification restores the production anchor through an exit trap.
- The project is a network leak-prevention layer, not an account-security guarantee.

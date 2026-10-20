# Host Gate

A Velocity plugin that limits Minecraft status pings and logins based on the
hostname sent in the client's handshake.

By default, Host Gate allows hostnames configured in Velocity's
`[forced-hosts]` section. Additional hostnames and IP addresses can be allowed
through its configuration.

Rejected status pings receive no Minecraft status response. Rejected logins are
silently closed by default.

## Requirements

* Java 25
* Velocity 4.x

## Build

```sh
mvn clean package
```

The built plugin is available in `target/`.

Copy the JAR to Velocity's `plugins/` directory, remove any older Host Gate
JARs, and restart the proxy.

## Configuration

Host Gate creates its configuration automatically on first start:

```text
plugins/hostgate/config.toml
```

The generated configuration documents all available options.

Set `logAllowedPings = true` in `plugins/hostgate/config.toml` to log status
pings that Host Gate allows. It defaults to `false` because public pings can
produce a lot of log entries. This records the decision made by Host Gate;
another plugin may still change the final response.

To suppress selected Host Gate request messages, add regular expressions to
`logFilters`. If any expression matches a message, that message is not logged.
Each expression is checked with substring matching
against the complete message, including its source address, handshake host,
and username (for rejected logins):

```toml
logRejectedConnections = true
logRejectedPings = true
logAllowedPings = true
logFilters = [
    "(?i)username 'matscan'",
    '151[.]115[.]73[.]107'
]
```

Replace the example address with the one observed in your own logs. Existing
configuration files can add the new array directly; omitted arrays are empty.
Run `/hostgate reload` after editing. Patterns are compiled at reload time;
if the list has an invalid expression, Host Gate warns and disables log
filtering without changing its connection policy. TOML literal strings
(single quotes) are convenient for regexes with backslashes. These settings
only affect Host Gate's log messages. Velocity, other plugins, and backend
servers may write their own logs for the same connection.

Host Gate snapshots normalized Velocity forced hosts when loading its policy,
so it does not need to scan and normalize that list for each ping or login.
`/hostgate reload` and `/velocity reload` refresh the snapshot. Existing
configuration files work without adding the new setting.

Allowed entries can include exact hostnames, exact IPv4/IPv6 addresses, and
single-level DNS wildcards such as `*.example.com`.

## Security considerations

Host Gate checks the hostname claimed by the Minecraft client. It makes generic
IP-based Minecraft scanning less useful, but it is not an authentication
mechanism.

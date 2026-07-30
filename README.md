# SMTP Client - Ecko Std Lib Package

An SMTP client (RFC 5321) for [Ecko](https://ecko.sh), written in Ecko. The
protocol is CRLF lines over `std.net`'s raw sockets (`connect` /
`connect_tls`, `send`, `recv_until`), with `net.starttls` upgrading the live
socket in place - no native code. AUTH PLAIN rides `std.encoding`'s base64.

## Install

```bash
ecko get github.com/ecko-lang/smtp-client
```

`ecko get` vendors the package under
`./vendor/github.com/ecko-lang/smtp-client/` and pins a file-tree hash in
`ecko.sum`.

`ecko get` records this dependency under the alias `smtp-client`, which
isn't a valid import name (hyphens aren't allowed in Ecko identifiers). Alias
it to `smtp` in your `ecko.json` - this also grants the network capability
the client needs:

```json
{
  "dependencies": {
    "smtp": {
      "path": "github.com/ecko-lang/smtp-client",
      "version": "v0.9.5",
      "grant": ["net"]
    }
  }
}
```

```ecko
import smtp
```

## Use

```ecko
import smtp

smtp.send(
    {
        host: "smtp.example.com",
        port: 587,                     # 587 -> STARTTLS, 465 -> TLS, else plain
        user: "me@example.com",
        pass: secret("app-password"),  # a `secret` is revealed only for AUTH
    },
    {
        from: "Me <me@example.com>",
        to: ["you@example.com"],       # a single string works too
        subject: "hello",
        body: "Sent from Ecko.",
    },
)
# -> { accepted: ["you@example.com"], reply: "2.0.0 queued as ..." }
```

### The mock transport

`{ mock: true }` skips the network entirely and returns the envelope plus the
built RFC 5322 message - deterministic, so examples and tests run offline:

```ecko
out = smtp.send({ mock: true }, { from: "a@x", to: "b@y", subject: "s", body: "hi" })
out.data    # the exact message a live send would deliver
```

## Server options

| field | meaning | default |
|-------|---------|---------|
| `host` | relay hostname | required (unless `mock`) |
| `port` | TCP port | 25 / 587 (starttls) / 465 (tls) |
| `tls` | `"none"` \| `"starttls"` \| `"tls"` | inferred from `port` |
| `user` / `pass` | AUTH PLAIN credentials (`pass` may be a `secret`) | no auth |
| `verify` | verify the server certificate | `true` |
| `helo` | EHLO name | `"localhost"` |
| `mock` | mock transport, no network | `false` |

## Message fields

| field | meaning |
|-------|---------|
| `from` | sender - `"a@x"` or `"Name <a@x>"` (envelope uses the bare address) |
| `to` | one recipient string or a list |
| `subject` | subject line |
| `body` | plain text; line endings normalized, leading dots escaped |
| `date` | optional `Date:` header; stamped at send time if omitted |
| `headers` | optional map of extra headers (emitted sorted by name) |

Failures throw `{ kind: "smtp", code, message }` - match on `code` for server
rejections (`535` bad credentials, `550` mailbox refused, ...). Error text
never includes credentials.

## Testing

```bash
ecko test               # offline: message building, dot-stuffing, AUTH
                        # encoding, reply parsing, mock transport
ecko example.ecko       # offline: the mock transport end to end
ecko example_live.ecko  # a real delivery (SMTP_HOST/USER/PASS/FROM/TO env)
```

## License

MIT

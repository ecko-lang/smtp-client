# smtp-client

## `addr_of(s)`

addr_of("Ada <ada@x>") -> "ada@x"; a bare address passes through trimmed.

## `parse_reply_line(line)`

One reply line -> { code, more, text }. "250-x" continues a multiline
reply; "250 x" (or a bare "250") ends it.

## `dot_stuff(text)`

Normalize line endings to CRLF and escape leading dots (RFC 5321 §4.5.2),
so a body line of "." can't terminate the DATA phase early.

## `auth_plain(user, pass)`

The AUTH PLAIN initial response: base64("\0user\0pass").

## `build_message(msg)`

build_message(msg) -> RFC 5322 text (CRLF line endings): From / To /
Subject [/ Date] / extra headers (sorted by name) / MIME headers, a blank
line, then the body. msg: { from, to (string|list), subject, body,
date?, headers? }.

## `read_reply(sock)`

read_reply(sock) -> { code, lines }, collecting a full (possibly
multiline) server reply.

## `expect_code(sock, want, label)`

Require the next reply to carry `want`, or throw { kind: "smtp", code }.

## `command(sock, line, want)`

Send one command line and require a reply code. Errors are labelled with
the command VERB only, so AUTH credentials never appear in error text.

## `send(server, msg)`

send(server, msg) -> { accepted, reply }
  server: { host, port?, tls?, user?, pass?, verify?, helo?, mock? }
  msg:    { from, to (string|list), subject, body, date?, headers? }

The password may be a `secret` - it is revealed only for the AUTH exchange.

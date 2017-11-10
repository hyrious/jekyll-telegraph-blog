---
title: Build APIs without any servers
---

- [RunKit](https://runkit.com) (server)
- [jsonbin](https://jsonbin.org) (storage)

#### TL;DR

Treat RunKit as a springboard.

[hyrious/playground](https://runkit.com/hyrious/playground) $$ \to $$
[playground-ex8yrl1662nc.runkit.sh][3]

(refresh to see the number++)

#### How it works

The [endpoint](https://runkit.io) service provided by RunKit could be
used as a small web service running some js(node) as long as someone
hit your *notebook's* unique url.

However, your running environment will be reset after 60 seconds to
prevent abuse.

Fortunately, we could found services providing other assets like storage.

With an auth token, you could save some data to jsonbin (and RunKit gives
[such place](https://runkit.com/settings/environment) to put our API
key, lucky!).

The best thing is, it's all free and under HTTPS.

#### Further more

You may know that glot.io could be used to [run][1] codes of [any][2]
language. What will happen if we combine RunKit with it?

[1]: https://github.com/prasmussen/glot-run/blob/master/api_docs/run.md
[2]: https://glot.io
[3]: https://playground-ex8yrl1662nc.runkit.sh

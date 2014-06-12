mackerel-agent-dev-vagrant
==========================

Development environment for [mackerelio/mackerel-agent](https://github.com/mackerelio/mackerel-agent),
using Vagrant to build and test on various environments.

Initialization
--------------

```
$ go get github.com/mackerelio/mackerel-agent
$ ./script/bootstrap
```

`rake test`
-----------

Runs mackerel-agent's `make test` on each Vagrant machines.

`rake watch`
------------

Watches local file modification to invoke `make test` on each Vagrant machines.

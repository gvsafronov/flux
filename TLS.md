TLS Support
===========

Getting Started
---------------

### Building

To build with TLS support you'll need OpenSSL development libraries (e.g.
libssl-dev on Debian/Ubuntu).

Run `make BUILD_TLS=yes`.

### Tests

To run fluidb test suite with TLS, you'll need TLS support for TCL (i.e.
`tcl-tls` package on Debian/Ubuntu).

1. Run `./utils/gen-test-certs.sh` to generate a root CA and a server
   certificate.

2. Run `./runtest --tls` or `./runtest-cluster --tls` to run fluidb and fluidb
   Cluster tests in TLS mode.

### Running manually

To manually run a Redis server with TLS mode (assuming `gen-test-certs.sh` was
invoked so sample certificates/keys are available):

    ./src/fluidb-server --tls-port 9470 --port 0 \
        --tls-cert-file ./tests/tls/fluidb.crt \
        --tls-key-file ./tests/tls/fluidb.key \
        --tls-ca-cert-file ./tests/tls/ca.crt

To connect to this Redis server with `fluidb-cli`:

    ./src/fluidb-cli --tls \
        --cert ./tests/tls/fluidb.crt \
        --key ./tests/tls/fluidb.key \
        --cacert ./tests/tls/ca.crt

This will disable TCP and enable TLS on port 9470. It's also possible to have
both TCP and TLS available, but you'll need to assign different ports.

To make a Replica connect to the master using TLS, use `--tls-replication yes`,
and to make fluidb Cluster use TLS across nodes use `--tls-cluster yes`.

Connections
-----------

All socket operations now go through a connection abstraction layer that hides
I/O and read/write event handling from the caller.

Note that unlike Redis, fluidb fully supports multithreading of TLS connections.

To-Do List
----------

- [ ] fluidb-benchmark support. The current implementation is a mix of using
  hiredis for parsing and basic networking (establishing connections), but
  directly manipulating sockets for most actions. This will need to be cleaned
  up for proper TLS support. The best approach is probably to migrate to hiredis
  async mode.
- [ ] fluidb-cli `--slave` and `--rdb` support.

Multi-port
----------

Consider the implications of allowing TLS to be configured on a separate port,
making fluidb listening on multiple ports:

1. Startup banner port notification
2. Proctitle
3. How slaves announce themselves
4. Cluster bus port calculation

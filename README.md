# go-statsd

Go statsd client library with zero allocation overhead, great performance and automatic
reconnects.

Client has zero memory allocation per metric sent:

* ring of buffers, each buffer is UDP packet
* buffer is taken from the pool, filled with metrics, passed on to the network delivery and
returned to the pool
* buffer is flushed either when it is full or when flush period comes (e.g. every 100ms)
* separate goroutines handle network operations: sending UDP packets and reconnecting UDP socket
* when metric is serialized, zero allocation operations are used to avoid `reflect` and temporary buffers

## Zero memory allocation

As metrics could be sent by the application at very high rate (e.g. hundreds of metrics per one request),
it is important that sending metrics doesn't cause any additional GC or CPU pressure. `go-statsd` is using
buffer pools and it tries to avoid allocations while building statsd packets.

## Reconnecting to statsd

With modern container-based platforms with dynamic DNS statsd server might change its address when container
gets rescheduled. As statsd packets are delivered over UDP, there's no easy way for the client to figure out
that packets are going nowhere. `go-statsd` supports configurable reconnect interval which forces DNS resolution.

While client is reconnecting, metrics are still processed and buffered.

## Dropping metrics

When buffer pool is exhausted, `go-statsd` starts dropping packets. Number of dropped packets is reported via
`Client.GetLostPackets()` and every minute logged using `log.Printf()`. Usually packets should never be dropped,
if that happens it's usually signal of enormous metric volume.

## Stastd server

Any statsd-compatible server should work well with `go-statsd`, [statsite](https://github.com/statsite/statsite) works
exceptionally well as it has great performance and low memory footprint even with huge number of metrics.

## Benchmark

Benchmark comparing several clients:

* https://github.com/alexcesaro/statsd/ (`Alexcesaro`)
* this client (`GoStatsd`)
* https://github.com/cactus/go-statsd-client (`Cactus`)
* https://github.com/peterbourgon/g2s (`G2s`)
* https://github.com/quipo/statsd (`Quipo`)
* https://github.com/Unix4ever/statsd (`Unix4ever`)

Benchmark results:

    BenchmarkAlexcesaro-8        	 3000000	       476 ns/op	       0 B/op	       0 allocs/op
    BenchmarkGoStatsd-8          	 5000000	       266 ns/op	       1 B/op	       0 allocs/op
    BenchmarkCactus-8            	 2000000	       626 ns/op	      11 B/op	       0 allocs/op
    BenchmarkG2s-8               	   50000	     20539 ns/op	     576 B/op	      21 allocs/op
    BenchmarkQuipo-8             	 1000000	      1508 ns/op	     383 B/op	       6 allocs/op
    BenchmarkUnix4ever-8         	 1000000	      1906 ns/op	     376 B/op	      18 allocs/op

## Origins

Ideas were borrowed from the following stastd clients:

* https://github.com/quipo/statsd (MIT License, https://github.com/quipo/statsd/blob/master/LICENSE)
* https://github.com/Unix4ever/statsd (MIT License, https://github.com/Unix4ever/statsd/blob/master/LICENSE)
* https://github.com/alexcesaro/statsd/ (MIT License, https://github.com/alexcesaro/statsd/blob/master/LICENSE)
* https://github.com/armon/go-metrics (MIT License, https://github.com/armon/go-metrics/blob/master/LICENSE)

## License

License is [MIT License](LICENSE).

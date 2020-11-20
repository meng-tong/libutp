# libutp - The uTorrent Transport Protocol library based on LEDBAT++.

This is a [LEDBAT++][ledbatpp] extension based on the original BitTorrent uTP
implementation of [LEDBAT][ledbat]. The preliminary extension includes the
following changes:

1. Modified slow start, and the termination condition of observed delay
exceeding 3/4 target delay (corresponding to 4.1 in
[LEDBAT++ IRTF draft][ledbatpp]).

2. Dynamic GAIN calculated from the relative relation between base and target
delay, for the purpose of slower rate increase (corresponding to 4.2 in LEDBAT++
IRTF draft).

3. Multiplicative decrease (corresponding to 4.3 in LEDBAT++ IRTF draft).

4. Initial and periodic slowdown for base delay probing (corresponding to 4.4 in
LEDBAT++ IRTF draft).

5. Target delay of 60 ms instead of 100 ms in LEDBAT (suggested in 4.5 in LEDBAT++
IRTF draft).

On basis of the above changes, when window decay happens due to loss/timeout, we
enforce the sender to either delay the scheduled slowdown, or cancel ongoing
slowdown, for correct behaviors.

Note that the LEDBAT++ IRTF draft also suggests replacing usage of one-way delay
with round-trip time, to be more compatible with TCP. We did not implement that
in this preliminary extension, which should not dramatically change LEDBAT++'s
performance.


## Building

As in the original uTP implementation, on non-windows platforms, building the
shared library is as simple as:

    make

## License

libutp is released under the [MIT][lic] license.

## Performance Benchmarking

To demonstrate the performance of our LEDBAT++ implementation, we compare it
with the original uTP LEDBAT codes, and Proteus-S, a scavenger protocol designed
by ourselves (codes publicly available [here][proteuscode], published in
[SIGCOMM 2020][proteuspaper]). Specifically, we let them compete with TCP CUBIC,
BBR, and Proteus-P (a primary protocol also proposed in the
[Proteus paper][proteuspaper]), respectively. An [Emulab][emulab] bottleneck
link with 50 Mbps bandwidth, 30 ms RTT, 4 BDF buffer is used. We include the
achieved figures of throughput trend under the [benchmarking][benchmarking]
folder.

First, the periodic slowdown distinguishes LEDBAT++ from LEDBAT when running
alone.

Then, compared with LEDBAT, LEDBAT++ is more conservative with a smaller target
delay and slower rate increase (combined with multiplicative decrease). That can
be validated by its lower throughput when competing with other primary protocols.

However, the slower rate increase also causes much longer ramp-up time, e.g.,
LEDBAT++ needs around 20 seconds to recover after the competing CUBIC flow
terminates. Such a slower rate increase in LEDBAT++ can also be observed in its
process of slowly taking up the bandwidth when competing with BBR.

In comparison, Proteus-S, as claimed in the [SIGCOMM paper][proteuspaper], is a
more robust scavenger, especially against latency-sensitive protocols such as
Proteus-P.

## Notes

We should strengthen that this LEDBAT++ implementation is still preliminary.
The value of all parameters and constants are adopted directly from the
[IRTF draft][ledbatpp]. Without performance tuning through extensive evaluation,
it may not have the same performance as Microsoft's own implementation in
Window Server operating system.


[ledbat]: http://datatracker.ietf.org/wg/ledbat/charter/
[bep29]: http://www.bittorrent.org/beps/bep_0029.html
[lic]: http://www.opensource.org/licenses/mit-license.php
[survey]: http://datatracker.ietf.org/doc/draft-ietf-ledbat-survey/
[ledbatpp]: https://tools.ietf.org/pdf/draft-irtf-iccrg-ledbat-plus-plus-01.pdf
[proteuspaper]: https://dl.acm.org/doi/pdf/10.1145/3387514.3405891
[benchmarking]: https://github.com/meng-tong/libutp/tree/master/benchmarking
[emulab]: https://www.emulab.net/portal/frontpage.php
[proteuscode]: https://github.com/PCCproject/PCC-Uspace/

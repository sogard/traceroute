# traceroute

This is a new modern implementation of the traceroute(8)
utility for Linux systems.

Traceroute tracks the route packets taken from an IP network on their
way to a given host. It utilizes the IP protocol's time to live (TTL)
field and attempts to elicit an ICMP TIME_EXCEEDED response from each
gateway along the path to the host.

Main features:
- Full support for both IPv4 and IPv6 protocols
- Several tracerouting methods, including:
  * UDP datagrams (including udplite and udp to particlular port)
  * ICMP ECHO packets (including dgram icmp sockets)
  * TCP SYNs (in general, any TCP request with various flags and options)
  * DCCP Request packets
  * Generic IP datagrams
- UDP methods do not require root privileges
- Ability to send several probe packets at a time
- Ability to compute a proper time to wait for each probe
- perform AS path lookups for returned addresses
- show ICMP extensions, including MPLS
- perform path MTU discovery automatically
- show guessed number of hops in backward direction
- command line compatible with the original traceroute
- and much more, see traceroute(8)

This code was written from the scratch, using some ideas of
Olaf Kirch's traceroute, the original implementation of Van Jacobson
(which was long used before) and some current BSD's ones.

This traceroute requires Linux kernel 2.6 and higher.

You can try to contact the author at <Dmitry at Butskoy dot name> .


Good tracerouting!

Dmitry Butskoy

# NEWS for 2.1.0:


The main significant change is new adaptive algorithm for waiting.

Traditional traceroute implementation always waited the whole timeout
(default 5.0 second) for any probe. But if we already have some replies from
the same hop, or even from some next hop, we can use the round trip time of
such a reply as a hint to determine the actual reasonable amount of time to
wait.

Now the `-w' option has a form of three (in general) float values separated
by a comma (or a slash): MAX_SECS,HERE,NEAR (last two are optional).
MAX_SECS specifies the maximum time to wait (as before), in any case.

The optional HERE specifies a factor to multiply the round trip time of an
already received response from the same hop.
The resulting value is used as a timeout for the probe, instead of (but no
more than) MAX_SECS. The optional NEAR specifies a similar factor for a
response from some next hop.
The time of the first found result is used in both cases.

First, we look for the same hop (of the probe which will be printed first
from now). If nothing found, then look for some next hop. If nothing found,
use MAX_SECS. If HERE and/or NEAR have zero values, the corresponding
computation is skipped.

HERE and NEAR are always set to zero if only MAX_SECS is specified. This
provides compatibility with previous versions. Thus, if your scripts uses
`-w SECS', then nothing changed for you, since the lonely SECS impliess
`-w SECS,0,0'

Defaults are 5.0 seconds for MAX_SECS, 3.0 times for HERE and 10.0 times
for NEAR. Preliminary tests show that these values seem good (else alter them
by cmdline).

Certainly, the new algorithm can lead to premature expiry (especially when
response times differ at times) and printing asterisk instead of a time.
Anyway, you can always switch this algorithm off, just by specifying `-w'
with the desired timeout only (for example, `-w 5').

We continue to wait whole MAX_SECS when one probe per time must be sent
(`--sport', `-P proto'), because it seems more harmful rather than helpful
to try to wait less in such cases.


Compatibility with traceroute-2.0.x:

"traceroute  -w 5"

Compatibility with previous implementations:

"traceroute  -N 1  -w 5"

(you can use any desired value for `-w', say `-w 0.5', just use *one* number).


Other changes in 2.1.0:

        *  Improve the main loop for better interactivity.

           Instead of waiting silently for maximum expiration time of probes
           in progress, use timeout of the first probe (which will be printed
           first from now) only.

	*  Hint people to use the system traceroute(8) instead of
	   tcptraceroute wrapper (by providing a stderr header).

	   The using of this wrapper is a little bit harmful, since it has
	   less possibilities and a little different set of options.

	   For those who are used to use tcptraceroute in cmdline,
	   just create a link with that name to the system traceroute.
	   When invoked as "tcp*", it then behaves as `traceroute -T'.
	   (The simple manual page added for this case in the wrapper subdir).

	   The original tcptraceroute had some options differ ("lpNSAE"),
	   but they was rare used. Most common "dnFifmqwst" was just the same.
	   Therefore it should be painless to use the system binary directly,
	   instead of the limited wrapper (which is still provided indeed).

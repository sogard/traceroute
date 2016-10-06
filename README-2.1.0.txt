NEWS for 2.1.0:


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

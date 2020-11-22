title = "Things that will go wrong in a distributed system"
date = 2020-11-22
%%%

A (very) incomplete list of things that will go wrong in any distributed system.

Feel free to [submit a PR](https://github.com/evuez/evuez.github.io/blob/master/posts/things-that-will-go-wrong-in-a-distributed-system.md) to add more failure cases to this list.

# Network

 - The network will be partitioned
 - Latency will grow more than expected
 - Timeouts will happen on nodes that are alive
 - Your network bandwidth is limited, and you will hit that limit

# Time

 - Clocks will go backward
 - Monotonic clocks will go backward [[1]](https://rachelbythebay.com/w/2020/10/20/ticktock/), [[2]](https://github.com/rust-lang/rust/pull/56988)
 - Clocks will be out of sync, by more than a few seconds sometimes
 - Your NTP server will die
 - You will have timezone issues

# Hardware

 - [HDDs will fail](http://static.googleusercontent.com/media/research.google.com/en//archive/disk_failures.pdf)
 - [RAMs will fail](https://arxiv.org/pdf/1901.03401.pdf)
 - [CPUs will fail](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/eurosys84-nightingale.pdf)

# Databases

 - Without [SSI](https://wiki.postgresql.org/wiki/SSI), you will have inconsistencies
 - Without [SSI](https://wiki.postgresql.org/wiki/SSI), you will lose data
 - Without a proper consensus, you will have more than one leader
 - Without [linearizability](https://en.wikipedia.org/wiki/Linearizability), clients will time travel
 - Without [2PC](https://en.wikipedia.org/wiki/Two-phase_commit_protocol), you will have inconsistencies

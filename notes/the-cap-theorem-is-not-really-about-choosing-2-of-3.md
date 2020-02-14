title = "The CAP theorem isn't really about \"choosing 2 of 3\""
%%%
When I first learned about the CAP therorem, I thought you _always_ had to choose 2 of these 3 guarantees:

 - **C**onsistency,
 - **A**vailability,
 - **P**artition tolerance

But that's not exactly right.

First, you can't really pick **C**onsistency and **A**vailability: if you forfeit **P**artition tolerance, you basically loose both **C**onsistency and **A**vailability because you can't keep them in case of network partition. So you actually have to choose between **CP** and **AP**.

But even that isn't completely true. Unless your network is unstable, you can have both **C**onsistency and **A**vailability _most of the time_. You don't need to choose between one of these as long as your network isn't partitioned.

And even in case of network partition, you don't always have to choose the same guarantee: it might be benificial in some cases to preserve **C**onsistency, while in some other it could be better to have **A**vailability.

title = "Head-of-line blocking"
%%%

Head-of-line blocking occurs when a small number of slow requests / packets holds up subsequent requests.

  - Optimizing slow requests is important as they might slow down a long line of fast ones,
  - Measuring response time on the client side can help catching these issues (as the processing time on the server wouldn't account for the time spent waiting for prior requests to complete).

[Head-of-line blocking on Wikipedia](https://en.wikipedia.org/wiki/Head-of-line_blocking).

## Single Thread

  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    12.21s     3.40s   18.09s    57.82%
    Req/Sec     0.02      2.97   394.00     99.99%
  11888 requests in 30.01s, 14.75MB read
  Socket errors: connect 0, read 0, write 0, timeout 135
Requests/sec:    396.11
Transfer/sec:    503.26KB

## Threading

  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    16.51s     4.64s   25.20s    59.19%
    Req/Sec   189.60     26.01   235.00     60.00%
  55029 requests in 30.01s, 68.28MB read
Requests/sec:   1833.75
Transfer/sec:      2.28MB

## Processes

  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     6.90s     2.17s   13.66s    65.09%
    Req/Sec   602.44    182.83     0.93k    68.75%
  192894 requests in 30.01s, 239.33MB read
Requests/sec:   6426.85
Transfer/sec:      7.97MB

## Async

  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.17ms  399.66us   4.91ms   69.05%
    Req/Sec     1.05k    70.34     1.33k    91.52%
  299994 requests in 30.00s, 372.21MB read
Requests/sec:   9999.91
Transfer/sec:     12.41MB

## Thread Pool

  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    16.37s     4.79s   25.25s    58.69%
    Req/Sec   164.00     36.90   250.00     70.00%
  53820 requests in 30.01s, 66.78MB read
Requests/sec:   1793.59
Transfer/sec:      2.23MB

## Process Pool

  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     6.89s     1.17s    9.61s    65.97%
    Req/Sec   794.77    167.74     1.02k    45.45%
  210358 requests in 30.01s, 261.00MB read
Requests/sec:   7010.63
Transfer/sec:      8.70MB

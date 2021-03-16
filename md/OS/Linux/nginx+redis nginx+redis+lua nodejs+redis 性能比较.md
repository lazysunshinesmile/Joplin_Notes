nginx+redis nginx+redis+lua nodejs+redis 性能比较

nginx+redis nginx+redis+lua nodejs+redis 性能比较

--------------------------------------redis直接测试性能--------------------------

PING_INLINE: 46082.95 requests per second
PING_BULK: 45045.04 requests per second
SET: 46511.63 requests per second
GET: 40983.61 requests per second
INCR: 47393.37 requests per second
LPUSH: 46511.63 requests per second
RPUSH: 45454.55 requests per second
LPOP: 40650.41 requests per second
RPOP: 39062.50 requests per second
SADD: 40322.58 requests per second
SPOP: 40160.64 requests per second
LPUSH (needed to benchmark LRANGE): 47169.81 requests per second
LRANGE_100 (first 100 elements): 28409.09 requests per second
LRANGE_300 (first 300 elements): 13513.51 requests per second
LRANGE_500 (first 450 elements): 11299.44 requests per second
LRANGE_600 (first 600 elements): 9276.44 requests per second
MSET (10 keys): 45045.04 requests per second

----------------------------------走js--------------------------------

-------------------set---------------------

Server Software:
Server Hostname:        localhost
Server Port:            12012

Document Path:          /Ryker/v1/DbOption.do
Document Length:        46 bytes

Concurrency Level:      2000
Time taken for tests:   5.318 seconds
Complete requests:      20000
Failed requests:        0
Total transferred:      2420000 bytes
Total body sent:        4320000
HTML transferred:       920000 bytes
Requests per second:    3760.81 [#/sec] (mean)
Time per request:       531.800 [ms] (mean)
Time per request:       0.266 [ms] (mean, across all concurrent requests)
Transfer rate:          444.39 [Kbytes/sec] received
                        793.30 kb/s sent
                        1237.69 kb/s total

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0  277 640.9      0    3010
Processing:    19   88  79.1     73    1716
Waiting:       18   88  79.1     73    1715
Total:         55  365 662.9     76    3497

Percentage of the requests served within a certain time (ms)
  50%     76
  66%     87
  75%    155
  80%   1058
  90%   1081
  95%   1270
  98%   3084
  99%   3107
100%   3497 (longest request)

---------------------get----------------------

Server Software:
Server Hostname:        localhost
Server Port:            12012

Document Path:          /Ryker/v1/DbOption.do
Document Length:        46 bytes

Concurrency Level:      2000
Time taken for tests:   5.274 seconds
Complete requests:      20000
Failed requests:        0
Total transferred:      2420000 bytes
Total body sent:        4320000
HTML transferred:       920000 bytes
Requests per second:    3792.12 [#/sec] (mean)
Time per request:       527.409 [ms] (mean)
Time per request:       0.264 [ms] (mean, across all concurrent requests)
Transfer rate:          448.09 [Kbytes/sec] received
                        799.90 kb/s sent
                        1247.99 kb/s total

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0  220 590.4      0    3006
Processing:    20   80  45.1     67     882
Waiting:       19   80  45.1     67     882
Total:         53  300 602.9     69    3308

Percentage of the requests served within a certain time (ms)
  50%     69
  66%     77
  75%     94
  80%    158
  90%   1086
  95%   1141
  98%   3070
  99%   3095
100%   3308 (longest request)

---------------------------------直接通过nginx------------------------

------------------------set-----------------------------

Server Software:        nginx/1.13.2
Server Hostname:        192.168.59.31
Server Port:            80

Document Path:          /redisTest.do?key=two&val=first
Document Length:        0 bytes

Concurrency Level:      2000
Time taken for tests:   2.452 seconds
Complete requests:      20000
Failed requests:        20000
   (Connect: 0, Receive: 0, Length: 19328, Exceptions: 672)
Total transferred:      3169792 bytes
HTML transferred:       289920 bytes
Requests per second:    8157.83 [#/sec] (mean)
Time per request:       245.163 [ms] (mean)
Time per request:       0.123 [ms] (mean, across all concurrent requests)
Transfer rate:          1262.63 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       46  112  24.3    113     181
Processing:    33  122  31.9    122     205
Waiting:        0   84  33.2     89     171
Total:        131  235  22.6    238     341

Percentage of the requests served within a certain time (ms)
  50%    238
  66%    242
  75%    245
  80%    248
  90%    258
  95%    264
  98%    275
  99%    284
100%    341 (longest request)

----------------------get----------------------------

Server Software:        nginx/1.13.2
Server Hostname:        192.168.59.31
Server Port:            80

Document Path:          /redisTest.do?key=two&val=first
Document Length:        15 bytes

Concurrency Level:      2000
Time taken for tests:   2.652 seconds
Complete requests:      20000
Failed requests:        960
   (Connect: 0, Receive: 0, Length: 480, Exceptions: 480)
Total transferred:      3201280 bytes
HTML transferred:       292800 bytes
Requests per second:    7542.37 [#/sec] (mean)
Time per request:       265.169 [ms] (mean)
Time per request:       0.133 [ms] (mean, across all concurrent requests)
Transfer rate:          1178.97 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       46  122  32.3    124     194
Processing:    29  128  33.8    128     232
Waiting:        0   76  42.8     69     201
Total:        182  250  22.5    251     306

Percentage of the requests served within a certain time (ms)
  50%    251
  66%    260
  75%    265
  80%    267
  90%    281
  95%    289
  98%    294
  99%    297
100%    306 (longest request)

----------------------------nginx + lua_redis ----------------------------------

------------------------------get-----------------------------------
Server Software:        nginx/1.13.2
Server Hostname:        127.0.0.1
Server Port:            80

Document Path:          /redisTest.do
Document Length:        22 bytes

Concurrency Level:      2000
Time taken for tests:   3.743 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      1710000 bytes
HTML transferred:       220000 bytes
Requests per second:    2671.87 [#/sec] (mean)
Time per request:       748.540 [ms] (mean)
Time per request:       0.374 [ms] (mean, across all concurrent requests)
Transfer rate:          446.18 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0  185 362.9      0    1003
Processing:    34  303 428.2     64    1975
Waiting:       33  303 428.2     63    1974
Total:         36  488 723.4     68    2973

Percentage of the requests served within a certain time (ms)
  50%     68
  66%    289
  75%    790
  80%    998
  90%   1832
  95%   2181
  98%   2641
  99%   2660
100%   2973 (longest request)

----------------------------set --------------------------------------

Server Software:        nginx/1.13.2
Server Hostname:        127.0.0.1
Server Port:            80

Document Path:          /redisTest.do
Document Length:        15 bytes

Concurrency Level:      2000
Time taken for tests:   3.302 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      1640000 bytes
HTML transferred:       150000 bytes
Requests per second:    3028.32 [#/sec] (mean)
Time per request:       660.433 [ms] (mean)
Time per request:       0.330 [ms] (mean, across all concurrent requests)
Transfer rate:          485.00 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0   88 256.0      0    1001
Processing:    15  359 650.0     53    2212
Waiting:       15  359 650.0     53    2212
Total:         15  447 770.0     53    2653

Percentage of the requests served within a certain time (ms)
  50%     53
  66%     63
  75%    353
  80%    576
  90%   1942
  95%   2181
  98%   2636
  99%   2639
100%   2653 (longest request)

-------------------------nginx + lua + redis的连接池-------------------------

--------------------------------set ----------------------------------
Server Software:        nginx/1.13.2
Server Hostname:        192.168.59.31
Server Port:            80

Document Path:          /nginx_redis_pool.do?command=set&key=key2&value=value2
Document Length:        59 bytes

Concurrency Level:      2000
Time taken for tests:   2.016 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      2080000 bytes
HTML transferred:       590000 bytes
Requests per second:    4960.75 [#/sec] (mean)
Time per request:       403.165 [ms] (mean)
Time per request:       0.202 [ms] (mean, across all concurrent requests)
Transfer rate:          1007.65 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0   49 153.0      0    1003
Processing:     5  193 224.9     97     856
Waiting:        3  193 224.8     96     856
Total:         12  242 304.0    101    1220

Percentage of the requests served within a certain time (ms)
  50%    101
  66%    115
  75%    132
  80%    390
  90%    699
  95%    984
  98%   1080
  99%   1208
100%   1220 (longest request)

-----------------------------------get ------------------------------

Server Software:        nginx/1.13.2
Server Hostname:        192.168.59.31
Server Port:            80

Document Path:          /nginx_redis_pool.do?command=get&value=value5
Document Length:        52 bytes

Concurrency Level:      2000
Time taken for tests:   5.036 seconds
Complete requests:      20000
Failed requests:        513
   (Connect: 0, Receive: 171, Length: 171, Exceptions: 171)
Total transferred:      3985629 bytes
HTML transferred:       1031108 bytes
Requests per second:    3971.10 [#/sec] (mean)
Time per request:       503.639 [ms] (mean)
Time per request:       0.252 [ms] (mean, across all concurrent requests)
Transfer rate:          772.82 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0   78 251.4      0    1004
Processing:    31  224 548.5     92    3349
Waiting:        0  195 470.5     91    3348
Total:         64  302 753.0     92    4347

Percentage of the requests served within a certain time (ms)
  50%     92
  66%    100
  75%    110
  80%    131
  90%    387
  95%   1295
  98%   4268
  99%   4282
100%   4347 (longest request)

------------------------------hget--------------------------------------

Server Software:        nginx/1.13.2
Server Hostname:        192.168.59.31
Server Port:            80

Document Path:          /nginx_redis_pool.do?command=hget&key=key1&filed=field1
Document Length:        67 bytes

Concurrency Level:      2000
Time taken for tests:   3.824 seconds
Complete requests:      20000
Failed requests:        0
Total transferred:      4320000 bytes
HTML transferred:       1340000 bytes
Requests per second:    5230.28 [#/sec] (mean)
Time per request:       382.389 [ms] (mean)
Time per request:       0.191 [ms] (mean, across all concurrent requests)
Transfer rate:          1103.26 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0   67 229.3      0    1005
Processing:     2  243 473.8     88    2077
Waiting:        1  243 473.8     88    2077
Total:          2  310 573.2     89    2140

Percentage of the requests served within a certain time (ms)
  50%     89
  66%    102
  75%    107
  80%    111
  90%   1623
  95%   1822
  98%   1975
  99%   2068
100%   2140 (longest request)

--------------------hset --------------------------------

Server Software:        nginx/1.13.2
Server Hostname:        192.168.59.31
Server Port:            80

Document Path:          /nginx_redis_pool.do?command=hset&key=key3&value=value3&field=filed3

Document Length:        75 bytes

Concurrency Level:      2000
Time taken for tests:   5.050 seconds
Complete requests:      20000
Failed requests:        0
Total transferred:      4480000 bytes
HTML transferred:       1500000 bytes
Requests per second:    3960.33 [#/sec] (mean)
Time per request:       505.008 [ms] (mean)
Time per request:       0.253 [ms] (mean, across all concurrent requests)
Transfer rate:          866.32 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0   71 256.0      0    3003
Processing:    29  198 441.2     94    3295
Waiting:       26  198 441.2     94    3295
Total:         73  269 627.7     95    4293

Percentage of the requests served within a certain time (ms)
  50%     95
  66%    101
  75%    116
  80%    136
  90%    473
  95%   1092
  98%   2721
  99%   4273
100%   4293 (longest request)

-------------------------------------hgetall------------------------------

Server Software:        nginx/1.13.2
Server Hostname:        192.168.59.31
Server Port:            80

Document Path:          /nginx_redis_pool.do?command=hgetall&key=key1
Document Length:        70 bytes

Concurrency Level:      2000
Time taken for tests:   4.778 seconds
Complete requests:      20000
Failed requests:        0
Total transferred:      4380000 bytes
HTML transferred:       1400000 bytes
Requests per second:    4185.44 [#/sec] (mean)
Time per request:       477.847 [ms] (mean)
Time per request:       0.239 [ms] (mean, across all concurrent requests)
Transfer rate:          895.13 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0  107 291.7      0    1039
Processing:    41  183 361.2     88    3280
Waiting:       41  182 361.1     87    3280
Total:         61  290 581.5     90    4277

Percentage of the requests served within a certain time (ms)
  50%     90
  66%    101
  75%    125
  80%    177
  90%    596
  95%   1489
  98%   2663
  99%   2752
100%   4277 (longest request)
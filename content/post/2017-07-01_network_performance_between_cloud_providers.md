---
title: "클라우드 서비스 네트워크 성능 측정"
date: 2017-07-01T15:59:00+09:00
---
<!--more-->
# 개요
오피스(판교)에서 Amazon과 Google 두 서비스에 구동된 VM까지의 네트워크 성능 측정

# 목적
메이저 클라우드 서비스 프로바이더들(Amazon, Google, Microsoft)은 현재 컨테이너 서비스를 도쿄 region에서만 지원하고 있음. 당분간 한국 IDC에 위치한 레거시 시스템과 연동이 필요한 상황에서, 도쿄 region에 시스템 구축시 latency와 bandwidth를 측정하여 쓸만한지 파악하려 함.

## 방법
* bandwidth : iperf3
  * 각 iperf3 결과는 3번 테스트 후 마지막 값.
* latency : ping
  * 각 ping 결과는 18~20번째 테스트 결과값.

## 대상
* AWS seoul region
* AWS tokyo region
* GCP tokyo region

# 결과

## 요약
* office -> AWS seoul EC2
```prettyprint
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-10.00  sec  86.3 MBytes  72.4 Mbits/sec    0             sender
[  4]   0.00-10.00  sec  85.3 MBytes  71.6 Mbits/sec                  receiver
```
```prettyprint
64 bytes from 52.79.95.212: icmp_seq=18 ttl=239 time=7.43 ms
64 bytes from 52.79.95.212: icmp_seq=19 ttl=239 time=6.44 ms
64 bytes from 52.79.95.212: icmp_seq=20 ttl=239 time=7.54 ms
```

* office -> AWS tokyo EC2
```prettyprint
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-10.00  sec  66.2 MBytes  55.6 Mbits/sec    0             sender
[  4]   0.00-10.00  sec  65.4 MBytes  54.8 Mbits/sec                  receiver
```
```prettyprint
64 bytes from 52.193.60.44: icmp_seq=18 ttl=235 time=42.7 ms
64 bytes from 52.193.60.44: icmp_seq=19 ttl=235 time=51.7 ms
64 bytes from 52.193.60.44: icmp_seq=20 ttl=235 time=42.0 ms
```

* office -> GCP tokyo VM
```prettyprint
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-10.00  sec  40.7 MBytes  34.2 Mbits/sec    0             sender
[  4]   0.00-10.00  sec  39.7 MBytes  33.3 Mbits/sec                  receiver
```
```prettyprint
64 bytes from 35.187.218.131: icmp_seq=18 ttl=60 time=35.3 ms
64 bytes from 35.187.218.131: icmp_seq=19 ttl=60 time=45.0 ms
64 bytes from 35.187.218.131: icmp_seq=20 ttl=60 time=40.8 ms
```

## 상세
* office -> AWS seoul EC2
  * type : t2.micro
  * connect : ssh ec2-user@52.79.95.212
  * test : iperf3 -c 52.79.95.212

```prettyprint
Connecting to host 52.79.95.212, port 5201
[  4] local 10.0.2.15 port 49056 connected to 52.79.95.212 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-1.00   sec  10.5 MBytes  88.2 Mbits/sec    0   78.4 KBytes
[  4]   1.00-2.00   sec  7.38 MBytes  61.9 Mbits/sec    0   78.4 KBytes
[  4]   2.00-3.00   sec  8.70 MBytes  73.0 Mbits/sec    0   78.4 KBytes
[  4]   3.00-4.00   sec  9.31 MBytes  78.1 Mbits/sec    0   78.4 KBytes
[  4]   4.00-5.00   sec  9.56 MBytes  80.1 Mbits/sec    0   78.4 KBytes
[  4]   5.00-6.00   sec  5.76 MBytes  48.3 Mbits/sec    0   78.4 KBytes
[  4]   6.00-7.00   sec  6.75 MBytes  56.6 Mbits/sec    0   78.4 KBytes
[  4]   7.00-8.00   sec  8.55 MBytes  71.7 Mbits/sec    0   78.4 KBytes
[  4]   8.00-9.00   sec  9.68 MBytes  81.2 Mbits/sec    0   78.4 KBytes
[  4]   9.00-10.00  sec  10.1 MBytes  84.8 Mbits/sec    0   78.4 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-10.00  sec  86.3 MBytes  72.4 Mbits/sec    0             sender
[  4]   0.00-10.00  sec  85.3 MBytes  71.6 Mbits/sec                  receiver

iperf Done.
```

* office -> AWS tokyo EC2
  * type : t2.micro
  * connect : ssh ec2-user@ec2-52-193-60-44.ap-northeast-1.compute.amazonaws.com
  * test : iperf3 -c 52.193.60.44

```prettyprint
Connecting to host 52.193.60.44, port 5201
[  4] local 10.0.2.15 port 53476 connected to 52.193.60.44 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-1.00   sec  4.70 MBytes  39.4 Mbits/sec    0   65.6 KBytes
[  4]   1.00-2.00   sec  4.87 MBytes  40.8 Mbits/sec    0   68.4 KBytes
[  4]   2.00-3.00   sec  7.08 MBytes  59.4 Mbits/sec    0   68.4 KBytes
[  4]   3.00-4.00   sec  7.08 MBytes  59.3 Mbits/sec    0   68.4 KBytes
[  4]   4.00-5.00   sec  7.08 MBytes  59.4 Mbits/sec    0   68.4 KBytes
[  4]   5.00-6.00   sec  7.26 MBytes  60.9 Mbits/sec    0   68.4 KBytes
[  4]   6.00-7.00   sec  6.98 MBytes  58.6 Mbits/sec    0   68.4 KBytes
[  4]   7.00-8.00   sec  7.08 MBytes  59.4 Mbits/sec    0   68.4 KBytes
[  4]   8.00-9.00   sec  7.17 MBytes  60.1 Mbits/sec    0   68.4 KBytes
[  4]   9.00-10.00  sec  6.95 MBytes  58.3 Mbits/sec    0   68.4 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-10.00  sec  66.2 MBytes  55.6 Mbits/sec    0             sender
[  4]   0.00-10.00  sec  65.4 MBytes  54.8 Mbits/sec                  receiver

iperf Done.
```

* office -> GCP tokyo
  * type : f1-micro
  * test : iperf3 -c 35.187.218.131

```prettyprint
Connecting to host 35.187.218.131, port 5201
[  4] local 10.0.2.15 port 57998 connected to 35.187.218.131 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-1.00   sec  4.24 MBytes  35.6 Mbits/sec    0   65.6 KBytes
[  4]   1.00-2.00   sec  4.14 MBytes  34.7 Mbits/sec    0   65.6 KBytes
[  4]   2.00-3.00   sec  4.26 MBytes  35.7 Mbits/sec    0   65.6 KBytes
[  4]   3.00-4.00   sec  4.04 MBytes  33.9 Mbits/sec    0   65.6 KBytes
[  4]   4.00-5.00   sec  3.98 MBytes  33.4 Mbits/sec    0   65.6 KBytes
[  4]   5.00-6.00   sec  4.66 MBytes  39.1 Mbits/sec    0   65.6 KBytes
[  4]   6.00-7.00   sec  3.98 MBytes  33.4 Mbits/sec    0   65.6 KBytes
[  4]   7.00-8.00   sec  3.55 MBytes  29.8 Mbits/sec    0   65.6 KBytes
[  4]   8.00-9.00   sec  3.77 MBytes  31.6 Mbits/sec    0   69.9 KBytes
[  4]   9.00-10.00  sec  4.10 MBytes  34.4 Mbits/sec    0   69.9 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-10.00  sec  40.7 MBytes  34.2 Mbits/sec    0             sender
[  4]   0.00-10.00  sec  39.7 MBytes  33.3 Mbits/sec                  receiver

iperf Done.
```
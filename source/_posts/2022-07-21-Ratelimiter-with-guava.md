---
title: "Guava Ratelimiter"
description: ""
categories: "2021-07"
tags: [限流,Guava,Ratelimiter]
---
    

- [[Github]]链接 [github](https://github.com/google/guava/blob/master/guava/src/com/google/common/util/concurrent/SmoothRateLimiter.java)  
- 在 [[Sentinel]]中有借鉴了这里的[实现](https://github.com/alibaba/Sentinel/wiki/%E9%99%90%E6%B5%81---%E5%86%B7%E5%90%AF%E5%8A%A8)  
- 传统的第一印象做法  
  - 记录上一次允许请求的时间戳，确保在新请求进来的时候，确保距离上一次允许请求已经过了 1/QPS 时间。  
    - 假如一个QPS=5的限制，在上一次允许的请求时间是t1 的i那个況下，下一次允许的请求必须要在t1+200ms (1/5) 时间后才能允许。如果上一次的请求允许时间是在100ms 之前，那么新的请求必须要再等100ms 后才能放行  
    - 在这个QPS 下，如果有15个新请求进来，那么必须要等3s (200ms * 15)才能放行完这15个请求  
      - 假如这个ratelimiter 很久没有请求进来，而突然间来了5个请求呢？也要200ms * 5 = 1s 才放行完？  
      - 这种情况怎么处理呢？  
- Guava 里面的实现  
  - 针对上面的问题，[[Guava]] 里面提出一个`past underutilization`，过去的低利用率。  
    - 在一个限流器里面，在它过去一段时间没有流量进来的情况下，当有流量突然进来了，过去的时间实际上限流器是没有起作用的，那么它是否可以被利用起来呢？从利用上来讲它可能是无成本的，因为时间都已经过去了。如果可以那么突然进来的流量就可以马上予以同行(假设没有超过限定值)  
    - 它这里提出了下面两个维度  
      - `stored permits`：在限流器过去一段时间没有被使用时，它可以累积一些令牌，等流量进来后，可以马上放行  
      - `fresh permits`: 正常饱和流量下可以获得的令牌，它的使用会让限流器把时间往前走n*(1/QPS)  
    - 例子  
      -  
        > For a RateLimiter that produces 1 token per second, every second that goes by with the RateLimiter being unused, we increase storedPermits by 1. Say we leave the RateLimiter unused for 10 seconds (i.e., we expected a request at time X, but we are at time X + 10 seconds before  a request actually arrives; this is also related to the point made in the last paragraph), thus storedPermits becomes 10.0 (assuming maxStoredPermits >= 10.0). At that point, a request of acquire(3) arrives. We serve this request out of storedPermits, and reduce that to 7.0 (how this is translated to throttling time is discussed later). Immediately after, assume that an acquire(10) request arriving. We serve the request partly from storedPermits, using all the remaining 7.0 permits, and the remaining 3.0, we serve them by fresh permits produced by the rate limiter.  

    - 一般来说，我们计算QPS是这样的 `rate=permits/times`，在一定时间内获得的令牌除以时间。那么速率就是`1/rate=times/permits`。那么`1/rate(times/permits)` 乘以令牌数就是**时间**了。这里的意思就是针对过去累积的令牌数来说的。比如说累积的令牌数是100,那么它消耗到只剩10 需要耗时多少，一旦消耗完成(有足够饱和的持续的流量进来)，限流器就会正常工作，就是以指定的QPS 释放令牌。  
    - 在流量稳定的情况下，一个`1QPS` 的限流器，我们可以知道它将会要3s 才能释放3个新的令牌。那么使用`stored permits`意味这什么呢？  
      - 假如我们关注的是过去的低利用率，那么在`stored permits` 有10 个的情况下，一个`acquire(7)` 请求来了，则应该就马上通行的。因为这10 个令牌是过去积累下来的，等于是免费的。  
      - 假如我们关注的是overflow, 那么我们就不能马上放行这7个令牌了。  
      - **这里就需要有一个地方处理这个转换的过程**  
    - 预热和冷却  
      - 这里把它拆分成两个过程对于后面的那个图形更好理解。  
        - 预热  
          - 在限流器刚开始时(或者很长一段时间没有流量进来)，开始有流量进来了，`store permits` 是有一定的积累，那么需要以慢一点的速度释放令牌。比如，在一个2QPS 的限流器稳定情况下，应该每个令牌获取的时间间隔是`1/QPS =1/2=500ms`，那么在这个阶段的间隔应该要调整，例如变成 `500ms * 3 = 1.5s/个` 这里的3 可以理解为一个冷却因子。这个速率保持的时间应该就是我们要指定的`warnupPeriod`  
        - 冷却  
          - 在流量稳定进入一段时间后，开始慢慢变少，直至没有流量。这个过程就是预热的逆过程。一旦流量变少，即意味着每个令牌过去的间隔就会变长，就会进入低利用率的状态，那么`stored permits` 就会增加。  
        - 这两个过程实际上是一致的。意思是说通过积分的方式去保证，`stored permits` 的变化，比如说由10 -> 7(预热)和7 -> 10(冷却)所需要的时间是一致的。这里同时还隐含着在这个变化过程中，`acquire(1)` ,`acquire(1)`,`acquire(1)`和`acquire(3)` 产生的效果(时间)是一致的。  
        - 限流器就会在这两个过程中动态进行。  
    -  
  - 概念  
    - `thresholdPermits`: 临界令牌数，当存储的令牌数到达这个值时就回到正常的速率，问题是这个值怎么算出来？**两条线的相交点**  
    - `stableIntervalMicros`:稳定速率  
    - `coldIntervalMicros`: 冷却速率  
    - `coldFactor`：冷却因子，怎么去理解这个概念呢？  
    - 两种限流器  
      - SmoothWarmingUp  
        - 带有预热时间的限流器  
        -  
          ```
                         * This implements the following function where coldInterval = coldFactor * stableInterval.
                         *
                         *          ^ throttling
                         *          |
                         *    cold  +                  /
                         * interval |                 /.
                         *          |                / .
                         *          |               /  .   <-- "warmup period" is the area of the trapezoid between
                         *          |              /   .       thresholdPermits and maxPermits
                         *          |             /    .
                         *          |            /     .
                         *          |           /      .
                         *   stable +----------/  WARM .
                         * interval |          .   UP  .
                         *          |          . PERIOD.
                         *          |          .       .
                         *        0 +----------+-------+--------------> storedPermits
                         *          0 thresholdPermits maxPermits
          ```
        - 这个图要分左右两个方向看。其中横坐标是`stored permits` 积累的令牌，越往右利用率越低。纵坐标是[[1]]/QPS，就是速率的意思代表每生产一个令牌需要花费多长时间  
          - 从右往左  
            - 开始时就是上面说的预热过程，在到稳定过程，随着时间走，直到`stored permits` 消耗完  
          - 从左往右  
            - 就是开始冷却的过程慢慢地`stored permits` 开始积累，直到达到最大值(此时没有流量进来)  
        - 从速率的角度去理解这个图，由0到`maxPermits`的速度应该是`maxPermits/warnupPeriod`  
        - `thresholdPermits` 的计算  
          - 预热的时长是一个比较短的时间  
        -  
      - SmoothBursty  
  - 调用流程  
    - 首先通过`acquire(n)`获取n 个令牌，会返回需要等待的时间，如果需要等待(`microToWait` 大于0)则进行线程`sleep`，否则就直接获取到令牌  
      -  
        ```
                    public double acquire(int permits) {
                    //计算获取令牌需要等待的时间
                      long microsToWait = reserve(permits);
                      //如果microsToWait > 0，则进行等待
                      stopwatch.sleepMicrosUninterruptibly(microsToWait);
                      //返回等待了的时间，转换成秒。
                      return 1.0 * microsToWait / SECONDS.toMicros(1L);
                    }
        ```
      - `reserve(int permits)`  
        -  
          ```
                        final long reserve(int permits) {
                        //检查permit,不允许小于0
                          checkPermits(permits);
                          synchronized (mutex()) {
                          //获取锁后预约令牌。因为这里会把`nextFreeTicketMicros` 往前拨
                            return reserveAndGetWaitLength(permits, stopwatch.readMicros());
                          }
                        }
          ```
    - `reserveAndGetWaitLength(int permits, long nowMicros)`  
      -  
        ```
                  final long reserveAndGetWaitLength(int permits, long nowMicros) {
                      //获取到permits 可以授予的时间戳
                      long momentAvailable = reserveEarliestAvailable(permits, nowMicros);
                      //这里返回需要等待的时间，两个时间戳相减，如果可以授予的时间戳在当前时间之后 （momentAvailable - nowMicros 大于 0），则需要等待
                      return max(momentAvailable - nowMicros, 0);
                    }
        ```
    - `resync(long nowMicros)`  
      -  
        ```
                    void resync(long nowMicros) {
                      // 里面的实现不是通过定期更新nextFreeTicketMicros，而是
                      //每次在获取时重新计算令牌的消耗情况
                      //当前时间在nextFreeTicketMicros 之后，说明限流器已经有一段时间没有被使用
                      //故需要更新时间指针
                      if (nowMicros > nextFreeTicketMicros) {
                      //冷却是按照一定的速率去进行的,每个不同限流器不一样
                      //warmup：warmupPeriodMicros / maxPermits
                      //bursty：stableInterval
                      //通过时间和速率的关系，计算出过去低利用率下可以利用的令牌
                        double newPermits = (nowMicros - nextFreeTicketMicros) / coolDownIntervalMicros();
                        storedPermits = min(maxPermits, storedPermits + newPermits);
                        nextFreeTicketMicros = nowMicros;
                      }
                    }
        ```
    - `reserveEarliestAvailable(int requiredPermits, long nowMicros)`  
      -  
        ```
                    final long reserveEarliestAvailable(int requiredPermits, long nowMicros) {
                      resync(nowMicros);//更新限流器的状态
                      //当前限流器可以获得令牌的时间戳，前面acquire 中会用它和当前时间做差值来决定是否需要等待
                      long returnValue = nextFreeTicketMicros;
                      //可以使用的storedPermits,最多能用当前积累的数量，不能超过。
                      double storedPermitsToSpend = min(requiredPermits, this.storedPermits);
                      //如果上面累积的令牌不够，计算需要多少个新令牌
                      double freshPermits = requiredPermits - storedPermitsToSpend;
                      //计算需要等待的时间，由两部分组成，一部分是使用积累的令牌，一部分是新的令牌
                      long waitMicros =
                          storedPermitsToWaitTime(this.storedPermits, storedPermitsToSpend)
                              + (long) (freshPermits * stableIntervalMicros);
                    //更新下一次令牌可以获得的时间戳
                      this.nextFreeTicketMicros = LongMath.saturatedAdd(nextFreeTicketMicros, waitMicros);
                      //减掉使用了的累积的令牌（结果可能会少于0）
                      this.storedPermits -= storedPermitsToSpend;
                      return returnValue;
                    }
        ```
    - `storedPermitsToWaitTime(double storedPermits, double permitsToTake)`  
      不同限流器的实现是不一样的，burty 是直接返回0了。  
      -  
        ```
                      long storedPermitsToWaitTime(double storedPermits, double permitsToTake) {
                        //这里的permitsToTake：stored permits to spend,前面已经做了处理保证不会大于storedPermits
                        //超过thresholdPermits 的可用令牌数
                        double availablePermitsAboveThreshold = storedPermits - thresholdPermits;
                        long micros = 0;
                        // 如果积累的令牌数有超过thresholdPermits 的话，计算要使用这些令牌需要的时间
                        //也就是梯形的面积，permitsToTime这个方法就是通过 y = a*x +b 这样的函数来计算的。
                        if (availablePermitsAboveThreshold > 0.0) {
                          double permitsAboveThresholdToTake = min(availablePermitsAboveThreshold, permitsToTake);
                          // permitsToTime(availablePermitsAboveThreshold)这个计算的是梯形的下底（从右往左看）
                          // permitsToTime(availablePermitsAboveThreshold - permitsAboveThresholdToTake)计算的梯形上底
                          double length = permitsToTime(availablePermitsAboveThreshold)
                              + permitsToTime(availablePermitsAboveThreshold - permitsAboveThresholdToTake);
                          //梯形的面积 (a + b) * h / 2
                          micros = (long) (permitsAboveThresholdToTake * length / 2.0);
                          permitsToTake -= permitsAboveThresholdToTake;
                        }
                        // 超过thresholdPermits 的令牌，以稳定的速率计算然后转换成时间
                        micros += (stableIntervalMicros * permitsToTake);
                        return micros;
                      }
        ```
    -  
    - 一次性获取全部的`storedPermits` 和 通过饱和的流量`acquire(1)`... 多次获取直至`storedPermits`用完，这两者之间的区别是什么？  
      - 前者第一次获取时就会全部通过，更新了`nextFreeTicketMicros`到下一次等待时间  
      - 后者就是慢慢地每一个令牌消耗，那么这两者用完了`storedPermits`后的下一次可以获取到令牌的时间是否一致呢？理论上是一致的。这里计算他们的时间间隔就可以。  
      -  
  - 为什么  
    - 为什么使用积分去计算  
    - 面积是代表时间的  
    - 从`maxPermits` 到 `threshholdPermits` 的时间是`warmupPeriod`。从`thresholdPermits` 到 0 的时间是`warmupPeriod / 2`。这里的解释是说因为之前的`coldFactor` 硬编码为3,这样做是为了保持和原有的行为一直。可是为什么`coldFactor` 是3的情况下，时间为`warmupPeriod / 2`，如果把`coldFactor` 设为7呢？这个时间又是多少呢？  
    
- References  
  - [RateLimiter 解读](https://cidoliu.github.io/2021/02/24/RateLimiter%E8%A7%A3%E8%AF%BB/)  
  - [Analysis of Current Limiting RateLimiter](https://blog.katastros.com/a?ID=00700-9dbb94e7-08d5-4761-88e1-98f96fc8f266)  
  - [how the value of thresholdPermits is cacluated in smoothWarnmingUp](https://groups.google.com/g/guava-discuss/c/w2f8HvnE47Q/m/o6ellgqRAwAJ)  
  - [[[Guava]]-[[RateLimiter]]-warmup-clarification](https://stackoverflow.com/questions/35298991/guava-ratelimiter-warmup-clarification)  
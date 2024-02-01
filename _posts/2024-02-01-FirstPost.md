---
title: "Redlock 작업"
author: naina
categories: 
  - IT
tags: 
  - Redlock
  - Redis
date: 2024-02-01
sidebar_main: true
---

나의 첫 블로그 게시물은 레드락 관련이다.

현재 홈IoT Backend를 개발/운영하고 있는데
Client의 요청이 동시에 들어올 경우 처리를 못하는 경우가 생겼다.

이를 해결하기 위해 Redis에 분산락을 거는 방향으로 접근하였다.


[Redlock 이란]( https://redis.io/docs/manual/patterns/distributed-locks/ )


Redlock 코드를 구현하고 로컬에서 수행하였을 때,
문제없이 락이 걸리고 이에 따라 요청을 순차적으로 처리하는 것을 확인하였다.

이때 테스트용 Redis는 싱글로 구성이 되어있었다.

근데 구현한 코드를 개발환경에 반영하였을 때,
Lock을 점유하지 못하였다는 에러를 받았다.

    error msg : The operation was unable to achieve a quorum during its retry window.

처음에는 redlock init 을 설정할 때,
redis client 의 정보를 cluster 별로 다 적어야 하는데 그게 아니어서 에러가 난게 아닌가? 라고 접근하였다.
근데 redlock 자체가 cluster redis 환경에 맞춰 lock을 거는건데......좀 이상하지 않나?

어쨋든 정말 이 문제가 맞는지 확인하기 위해
cluster 환경으로 redis를 구성하였다.

이후 테스트를 했는데,, 음 문제없이 작동한다.

그렇다면 cluster와 single의 문제는 아니다.

그러다 개발환경에 설정된 redis의 user정보를 확인하였더니,
테스트를 하기 위해 만든 redis의 user정보와 달랐다.

그래서 user별 권한의 문제겠구나 접근하였다.

redis는 user별로 접근할 수 있는 key가 제한된다.
테스트로 만든거에는 default계정으로 "on ~* +@all" 이었고
개발환경 redis는 해당 계정의 "on ~계정:* +@all" 이었다.

그래서 redlock 걸 때, key를 아래와 같이 수정하였다.

ex) 계정 : xxxx

    const key = "xxxx:"+parts.slice(0, 3).join("/");

    let lock;
    try {
        lock = await context.redlock.acquire([key], timeout)
        context.publish(this.requestTopic, payload)

        if(lock){
          console.log("lock key : " + key + " , lock token : " + lock.value)
        }

        let result = await pEvent(context.emitter, this.responseTopic, {
            timeout: timeout,
        })

        return result
    } catch (e) {
        /* TODO : timeout error */
        console.error("error : " + e)
        return callback(e)
    } finally {
         if (lock) {
             await lock.release();
         }
    }

> 결론으론, 권한 문제가 맞았고 redlock key를 redis user정보에 맞춰 생성하였더니 lock획득을 정상적으로 했다.	
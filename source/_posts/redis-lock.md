---

title: 分布式锁（Redis实现版）

date: 2020-03-25 13:14

tags:
- Redis
- 分布式

categories: Redis

permalink: redis-lock

---

## 简介

这篇POST将会介绍一个典型的Redis分布式锁，以及使用Redis锁是碰到常见问题的解决方案



## 加锁

~~~java
public boolean tryLock(String resourceKey, String threadId) {
    // SET resourceKey threadId NX PX 30000
}
~~~

- `resourceKey`代表需要锁定的资源，一般是将资源类型和资源ID两个信息拼接起来，用来唯一指定那个资源，如`user-1001239123`

- `threadId`代表上锁的线程，解锁时必须提供自己上锁时提供的threadId，来确保自己上的锁只有自己能解开，`threadId`必须是集群中每台机器每个线程唯一，没有其他要求，所以用uuid即可。

- `NX`代表只在key不存在时， 才对key进行SET操作，这个参数确保了锁的互斥性
- `PX 30001`代表锁的超时时间，如果长时间不操作锁，往往代表上锁的机器已经挂了，应当通过key失效的方式解锁。



## 解锁



~~~java
public boolean tryRelease(String resourceKey, String threadId) {
		/*
        if redis.call('get', KEYS[1])==ARGV[1] then 
            return redis.call('del', KEYS[1])
        else
            return 0
        end
    */
}

~~~

- 使用LUA脚本来解锁，逻辑虽然是非原子的，但Redis会原子地执行LUA脚本
- key的value（即加锁是传入的threadId）必须与解锁API传入的threadId一致，确保自己上的锁只有自己能解
- 用户代码应当确保自己生成UUID不扩散到别的线程，否则别的线程也能解锁了

解锁的API应该设计成这样

~~~java

~~~

## 怎么实现可重入？

~~~java
private ThreadLocal<Multiset<String, Integer>> map;

public boolean tryLockWithReentrancy(String resourceKey, String threadId) {
    if (map.containKey(key)) {
        if (this.reflesh(resourceKey)) {
            map.put(resourceKey, map.get(resourceKey));
            return true;
        } 
    }

    if (this.tryLock(resourceKey, threadId)) {
        map.put(resourceKey, 1);
        return true;
    }
    return false;
}

private boolean reflesh(String resouceKey) {
   // PEXPIRE resourceKey 30000
}
~~~

~~~java
public boolean tryReleaseWithReentrancy(String resourceKey, String threadId) {
    int count = map.get(resourceKey);
    if (count == null) {
       return false;
    }
    if (count > 1) {
        map.put(resourceKey, count - 1);
        return true;
    }
    if (count == 1) {
        if (this.tryRelease(resourceKey, threadId)) {
            map.remove(resourceKey);
            return true;
        }
    }
    throw new IllegalStateException("impossible unless bug.");
}
~~~

- 这个map代表当前线程获取了多少次某个锁
- String代表resourceKey，Integer代表这个锁获取了几次



## key失效了业务代码还没执行完怎么办？

`tryLock`成功后将threadId交给一个用于监视的线程，监视线程每1/3失效时间轮询一次，当轮询到key即将失效时，刷新失效时间。




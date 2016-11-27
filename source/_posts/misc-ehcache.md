---
title: Ehcache
date: 2016-11-27 13:35:23
tags: Misc
---

Ehcache 使用起来很简单，是一个 `key/value` 容器，和 Map 很像，只不过功能比 Map 丰富，能够限定缓存中元素的个数，自动删除超时的元素，把元素持久化到硬盘等:

* 添加元素到缓存: `cache.put(new Element("username", "Biao"))`
* 从缓存获取元素: `cache.get("username")`
* 从缓存删除元素: `cache.remove("username")`
* CacheManager 管理多个 Cache，每个 Cache 里又管理多个缓存的元素 Element

<!--more-->

目录结构:

```
├── main
│   ├── java
│   │   ├── EhcacheHello.java
│   │   ├── EhcacheWithListener.java
│   │   ├── MyCacheEventListener.java
│   │   └── MyCacheEventListenerFactory.java
│   └── resources
│       └── ehcache.xml
└── test
    ├── java
    └── resources
```

## Maven 依赖

```groovy
compile 'net.sf.ehcache:ehcache:2.10.3'
```

## ehcache.xml

```xml
<ehcache updateCheck="false" name="fooCache">
    <!--默认的缓存配置-->
    <defaultCache maxElementsInMemory="10000"
                  eternal="false"
                  timeToIdleSeconds="120"
                  timeToLiveSeconds="120"
                  overflowToDisk="false"
                  diskPersistent="false"
                  diskExpiryThreadIntervalSeconds="120"/>

    <cache name="fox"
           maxElementsInMemory="100"
           eternal="false"
           timeToIdleSeconds="120"
           timeToLiveSeconds="120"
           overflowToDisk="false"
           diskPersistent="false"/>
</ehcache>
```

## 使用 EhCache

```java
import net.sf.ehcache.Cache;
import net.sf.ehcache.CacheManager;
import net.sf.ehcache.Element;

import java.net.URL;

public class EhcacheHello {
    /**
     * 使用 defaultCache 的配置创建缓存
     *
     * @param cacheManager
     */
    public static void createCacheWithDefaultConfiguration(CacheManager cacheManager) {
        cacheManager.addCache("buddha"); // 创建 Cache

        Cache cache = (Cache) cacheManager.getCache("buddha");
        cache.put(new Element("username", "Biao"));
        System.out.println(cache.get("username"));
    }

    /**
     * 获取配置中的缓存
     *
     * @param cacheManager
     * @param cacheName
     */
    public static void useConfiguredCache(CacheManager cacheManager, String cacheName) {
        Cache cache = (Cache) cacheManager.getCache(cacheName); // 获取 Cache

        cache.put(new Element("password", "S--S"));
        cache.put(new Element("username", "Viva"));
        cache.put(new Element("username", "Pandora"));
        System.out.printf("Size of cache %s: %d\n", cacheName, cache.getSize());

        cache.remove("username");
        cache.put(new Element("username", "Viva"));
    }

    public static void main(String[] args) {
        URL url = EhcacheHello.class.getResource("/ehcache.xml");
        CacheManager cacheManager = CacheManager.create(url);

        createCacheWithDefaultConfiguration(cacheManager);
        useConfiguredCache(cacheManager, "fox");

        System.out.println("缓存:");
        for (String name : cacheManager.getCacheNames()) {
            System.out.println(name);
        }

        // cacheManager.shutdown();
    }
}
```

输出:

```
[ key = username, value=Biao, version=1, hitCount=1, CreationTime = 1480225511239, LastAccessTime = 1480225511240 ]
Size of cache fox: 2
缓存:
buddha
fox
```

## Ehcache 的配置参数

| Attribute or Tag                | Description                              |
| ------------------------------- | ---------------------------------------- |
| name                            | 缓存名称                                     |
| maxElementsInMemory             | 缓存元素的最大数量                                |
| eternal                         | 对象是否永久有效，一但设置了，timeout 将不起作用 (eternal: 永恒的) |
| timeToIdleSeconds               | 设置对象在失效前的允许闲置时间（单位: 秒）。仅当 eternal=false 对象不是永久有效时使用，可选属性，默认值是 0，也就是可闲置时间无穷大。get(), put() 都会更新元素的最后访问时间，例如 timeToIdleSeconds=10，元素的 key 为 username, 在 10 秒内没有 get() 和 put() 操作的话，username 对应的元素就会过期，从缓存中删除，再次访问的时候得到的是 null |
| timeToLiveSeconds               | 设置对象在失效前允许存活时间（单位: 秒）。最大时间介于创建时间和失效时间之间。仅当 eternal=false 对象不是永久有效时使用，默认是 0，也就是对象存活时间无穷大。 例如 timeToLiveSeconds=10，则元素在创建 10 秒后不管有没有被访问，都会过期 |
| overflowToDisk                  | 当内存中对象数量超过 maxElementsInMemory 时，Ehcache 将会对象写到磁盘中 |
| diskSpoolBufferSizeMB           | 这个参数设置 DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区 |
| maxElementsOnDisk               | 硬盘最大缓存个数                                 |
| diskPersistent                  | 是否缓存虚拟机重启期数据 Whether the disk store persists between restarts of the Virtual Machine. The default value is false |
| diskExpiryThreadIntervalSeconds | 磁盘失效线程运行时间间隔，默认是120秒                     |
| memoryStoreEvictionPolicy       | 当达到 maxElementsInMemory 限制时，Ehcache 将会根据指定的策略去清理内存。默认策略是 LRU（最近最少使用）。你可以设置为 FIFO（先进先出）或是 LFU（较少使用） |
| clearOnFlush                    | 内存数量最大时是否清除                              |
| `<cacheEventListenerFactory>`   | 设置事件监听器，监听缓存中元素的添加，更新，删除等事件              |

## 自定义 CacheEventListener

添加，更新，删除元素等的时候都有相应的事件，可以自定义事件监听器来监听这些事件:

1. 实现接口 `CacheEventListener`，实现事件对应的方法
2. 实现接口 `CacheEventListenerFactory`，创建监听器
3. 配置文件里使用 `<cacheEventListenerFactory>` 注册事件监听器工厂
4. 事件发生的时候监听器里对应的方法会被自动调用

### ehcache.xml

设置缓存 fox 的 timeToIdleSeconds 为 3 秒，这样缓存里的元素只要一不使用，很快就过期了。

```xml
<ehcache updateCheck="false" name="fooCache">
    <!--默认的缓存配置-->
    <defaultCache maxElementsInMemory="10000"
                  eternal="false"
                  timeToIdleSeconds="120"
                  timeToLiveSeconds="120"
                  overflowToDisk="false"
                  diskPersistent="false"
                  diskExpiryThreadIntervalSeconds="120"/>

    <cache name="fox"
           maxElementsInMemory="100"
           eternal="false"
           timeToIdleSeconds="3"
           timeToLiveSeconds="120"
           overflowToDisk="false"
           diskPersistent="false">
        <cacheEventListenerFactory class="MyCacheEventListenerFactory"/>
    </cache>
</ehcache>
```

### MyCacheEventListener

```java
import net.sf.ehcache.CacheException;
import net.sf.ehcache.Ehcache;
import net.sf.ehcache.Element;
import net.sf.ehcache.event.CacheEventListener;

/**
 * 1. notifyElementRemoved(): 调用 remove() 删除元素时触发
 * 2. notifyElementExpired(): 元素超时时触发
 * 3. notifyElementEvicted(): 元素个数超过 maxElementsInMemory 被放到硬盘上时触发
 *
 * 注意: 这几个事件不会同时触发
 */
public class MyCacheEventListener implements CacheEventListener {
    @Override
    public void notifyElementRemoved(Ehcache cache, Element element) throws CacheException {
        System.out.println("notifyElementRemoved: " + element);
    }

    /**
     * put(), 但是缓存里没有，新创建
     * @param cache
     * @param element
     * @throws CacheException
     */
    @Override
    public void notifyElementPut(Ehcache cache, Element element) throws CacheException {
        System.out.println("notifyElementPut: " + element);
    }

    /**
     * put(), 但是已经存在缓存里，更新
     * @param cache
     * @param element
     * @throws CacheException
     */
    @Override
    public void notifyElementUpdated(Ehcache cache, Element element) throws CacheException {
        System.out.println("notifyElementUpdated: " + element);
    }

    /**
     * 注意: 元素过期不会自动触发, 下面 2 种情况时才会检查元素是否过期:
     *     1. 元素 x 有 get 操作时才检查它是否过期, 这时不会检查元素 y 是否过期
     *     2. 元素个数超过 maxElementsInMemory 时, 检查所有元素是否过期
     * @param cache
     * @param element
     */
    @Override
    public void notifyElementExpired(Ehcache cache, Element element) {
        System.out.println("notifyElementExpired: " + element);
    }

    @Override
    public void notifyElementEvicted(Ehcache cache, Element element) {
        System.out.println("notifyElementEvicted: " + element);
    }

    @Override
    public void notifyRemoveAll(Ehcache cache) {

    }

    @Override
    public void dispose() {

    }

    @Override
    public Object clone() throws CloneNotSupportedException {
        return null;
    }
}
```

### MyCacheEventListenerFactory

```java
import net.sf.ehcache.event.CacheEventListener;
import net.sf.ehcache.event.CacheEventListenerFactory;

import java.util.Properties;

public class MyCacheEventListenerFactory extends CacheEventListenerFactory {
    @Override
    public CacheEventListener createCacheEventListener(Properties properties) {
        return new MyCacheEventListener(); // 创建返回缓存的事件监听器
    }
}
```

### 使用 EhCache + 事件监听器

```java
import net.sf.ehcache.Cache;
import net.sf.ehcache.CacheManager;
import net.sf.ehcache.Element;

import java.net.URL;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class EhcacheWithListener {
    /**
     * 获取配置中的缓存
     * @param cacheManager
     * @param cacheName
     */
    public static void useConfiguredCache(CacheManager cacheManager, String cacheName) {
        Cache cache = (Cache) cacheManager.getCache(cacheName);

        cache.put(new Element("password", "S--S"));
        cache.put(new Element("username", "Viva")); // notifyElementPut()
        cache.put(new Element("username", "Pandora")); // notifyElementUpdated()
        System.out.printf("Size of cache %s: %d\n", cacheName, cache.getSize());

        cache.remove("username"); // 触发 notifyElementRemoved()
        cache.put(new Element("username", "Viva"));
    }

    /**
     * 即使元素过期了, 但是 Cache 不会马上就判断出它已经过期, 只有对其执行 get() 访问或者
     * Cache 中的元素数量大于 maxElementsInMemory 时才会检查元素是否过期.
     * 为了及时的发现元素过期了, 启动一个线程定期调用 evictExpiredElements() 检查出过期的元素,
     * 然后及时, 自动的把它们从缓存里删除.
     *
     * 某些场景下手动检测元素过期是很有必要的, 例如心跳检测数据是放到缓存里的, 当一定时间内没有接收到心跳数据时,
     * 即缓存里的心跳数据就会过期, 不过 EhCache 又不会及时自动检测过期的元素, 所以我们就有必要自己
     * 启动一个线程来检测出过期的数据然后使其从缓存里被删除.
     * @param cacheManager
     * @param cacheName
     */
    public static void checkExpiration(CacheManager cacheManager, String cacheName) {
        ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);
        scheduler.scheduleWithFixedDelay(new Runnable() {
            @Override
            public void run() {
                Cache cache = (Cache) cacheManager.getCache(cacheName);
                cache.get("username"); // 只有 password 会过期, 因为 username 的使用时间一直在更新
                cache.evictExpiredElements(); // 过期的元素会被自动从缓存删除
                System.out.printf("Size of cache %s: %d\n", cache.getName(), cache.getSize());
            }
        }, 1, 1, TimeUnit.SECONDS);
    }

    public static void main(String[] args) {
        URL url = EhcacheWithListener.class.getResource("/ehcache.xml");
        CacheManager cacheManager = CacheManager.create(url);

        useConfiguredCache(cacheManager, "fox");
        checkExpiration(cacheManager, "fox");

        // cacheManager.shutdown();
    }
}
```

输出:

```
notifyElementPut: [ key = password, value=S--S, version=1, hitCount=0, CreationTime = 1455261752745, LastAccessTime = 1455261752745 ]  
notifyElementPut: [ key = username, value=Viva, version=1, hitCount=0, CreationTime = 1455261752746, LastAccessTime = 1455261752746 ]  
notifyElementUpdated: [ key = username, value=Pandora, version=1, hitCount=0, CreationTime = 1455261752746, LastAccessTime = 1455261752746 ]  
Size of cache fox: 2  
notifyElementRemoved: [ key = username, value=Pandora, version=1, hitCount=0, CreationTime = 1455261752746, LastAccessTime = 1455261752746 ]  
notifyElementPut: [ key = username, value=Viva, version=1, hitCount=0, CreationTime = 1455261752751, LastAccessTime = 1455261752751 ]  
Size of cache fox: 2  
Size of cache fox: 2  
notifyElementExpired: [ key = password, value=S--S, version=1, hitCount=0, CreationTime = 1455261752745, LastAccessTime = 1455261752745 ]  
Size of cache fox: 1  
Size of cache fox: 1  
Size of cache fox: 1  
Size of cache fox: 1  
```


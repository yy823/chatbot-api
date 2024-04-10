# Redis

### Redis的操作

技术派将Redis的操作封装在RedisClient类中，其所有方法都是静态方法，意味着可以在任何地方操作Redis。

RedisClient类的内容如下：

基本操作：

-   register(RedisTemplate&lt;String, String>) : void
-   nullCheck(Object...) : void
-   valBytes(T) : byte[]
-   keyBytes(String) : byte[] []
-   keyBytes(List&lt;String>) : byte[] []
-   toObj(byte[], Class&lt;T>) : T
-   piplineAction() : PipelineActioin

对string的操作：

-   getStr(String) : String
-   setStr(String, String) : void
-   del(String) : void
-   expire(String, Long) : void
-   setStrWithExpire(String, String, Long) : Boolean

对hash的操作：

-   hGetAll(String, Class&lt;T>) : Map&lt;String, T>
-   hGet(String, String, Class&lt;T>) : T
-   hIncr(String, String, Integer) : Long
-   hDel(String, String) : Boolean
-   hSet(String, String, T) : Boolean
-   hMSet(String, Map&lt;String, T>) : void
-   hMGet(String, List&lt;String>, Class&lt;T>) : Map&lt;String, T>

对set的操作：

-   sIsMember(String, T) : Boolean
-   sGetAll(String, Class&lt;T>) : Set&lt;T>
-   sPut(String, T) : boolean
-   sDel(String, T) : void

对Zset的操作：

-   zIncrBy(String, String, Integer) : Double
-   zRankInfo(String, String) : ImmutablePair&lt;Integer, Double>
-   zScore(String, String) : Double
-   zRank(String, String) : Integer
-   zTopNScore(String, int) : List&lt;ImmutablePair&lt;String, Double>>

list操作：

-   lPush(String, T) : Long
-   rPush(String, T) : Long
-   lRange(String, int, int, Class&lt;T>) : List&lt;T>
-   lTrim(String, int, int) : void

**补充信息**：

-   其中，ImmutablePair&lt;L, R>是一个Java内部的、不常用的数据结构。

    -   其是一个单体数据对，其继承于Pair&lt;L, R>类。Pair&lt;L, R>类又实现了Map.Entry&lt;L, R>接口。

    -   ImmutablePair内部的数据不可变的。

    -   Pair类的默认实现是ImmutablePair。

-   RedisClient内部编写了一个内部类PipelineAction，用于封装多个操作为管道执行。同时使用了Java官方的BiConsumer，还创建了一个三参数的方法接口ThreeConsumer，用于传递参数。下面为其源码：

    ```java
    public static class PipelineAction {
        private List<Runnable> run = new ArrayList<>();
    
        private RedisConnection connection;
    
        public PipelineAction add(String key, BiConsumer<RedisConnection, byte[]> conn) {
            run.add(() -> conn.accept(connection, RedisClient.keyBytes(key)));
            return this;
        }
    
        public PipelineAction add(String key, String field, ThreeConsumer<RedisConnection, byte[], byte[]> conn) {
            run.add(() -> conn.accept(connection, RedisClient.keyBytes(key), valBytes(field)));
            return this;
        }
    
        public void execute() {
            template.executePipelined((RedisCallback<Object>) connection -> {
                PipelineAction.this.connection = connection;
                run.forEach(Runnable::run);
                return null;
            });
        }
    }
    @FunctionalInterface
    public interface ThreeConsumer<T, U, P> {
        void accept(T t, U u, P p);
    }
    ```

    RedisClient内部提供了一个工厂方法用于创建一个PipelineAction实例：

    ```java
    public static PipelineAction pipelineAction() {
        return new PipelineAction();
    }
    ```

-   RedisClient大部分方法的实现都基于RedisTemple的execute()方法，内部实现RedisCallback接口，其是一个方法接口，例如：

    ```java
    public static Double zIncrBy(String key, String value, Integer score) {
        return template.execute(new RedisCallback<Double>() {
            @Override
            public Double doInRedis(RedisConnection connection) throws DataAccessException {
                return connection.zIncrBy(keyBytes(key), score, valBytes(value));
            }
        });
    }
    ```

**吐槽一下**：我不说可能没人注意到，sPut()返回的boolean而不是Boolean。杀死强迫症！

###Redis的持久化 

在技术派中，Redis不再是一个简单的缓存工具，而是一个基于内存的快速的数据库，所以需要一定的持久化策略。

很遗憾，我没有找到关于Redis持久化的配置信息……但猜测应该是AOF混合持久化。

### Redis的集群

在技术派中，尝试使用Redis的集群来扩展更多的分布式内容，但是对应整体的业务来看，Redis集群只是保证了Redis的高可用特性，对于开发人员调用Redis接口来说是无感的。

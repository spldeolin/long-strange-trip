---
title: 常用lambda（Map相关）

date: 2018-12-17 18:17:12

updated: 2018-12-17 18:17:12

tags:
- lambda

categories: Java

permalink: lambda-map
---





## 将List映射到Map

~~~java
/*
 * 代码示例的注释用的都是“伪JSON”，仅仅是为了说明数据结构，下同
 */

// users [{id=3L, name=c, key=value1}, {id=2L, name=b, key=value2}, {id=1L, name=a, key=value3}]

Map<Long, User> usersById = users.stream().collect(Collectors.toMap(User::getId, one -> one));

// usersById {1={id=1, name=a, key=value3}, 2={id=2, name=b, key=value2}, 3={id=3, name=c, key=value1}}
~~~



其中，`Collectors.toMap`方法的第一个参数是作为Map的key来映射User对象的，key必须唯一，否则会倒是快速失败，抛出`java.lang.IllegalStateException: Duplicate key`异常



## 将List分组映射

可以看到，`Collectors.toMap`要求key必须唯一，所以他的返回值是`Map<?, User>`。
如果需要用非唯一的字段来映射，那可以换一个Collector

~~~java
// users [{id=3L, name=same, key=value1}, {id=2L, name=same, key=value2}, {id=1L, name=diff, key=value3}]

Map<String, List<User>> userGroupsByName = users.stream().collect(Collectors.groupingBy(User::getName));

// userGroupsByName {same=[{id=3, name=same, key=value1}, {id=2, name=same, key=value2}], diff=[{id=1, name=diff, key=value3}]}
~~~



## 将List分组映射到Guava Multimap

`Map<String, List<User>>`是一种很不优雅的返回类型，有的选的话，Deolin会使用`Multimap<String, User>`的方式来代替它

~~~java
Multimap<String, User> userGroupsByNameEx = users.stream().collect(MultimapCollectors.groupingBy(User::getName));
~~~



其中，`MultimapCollectors`需要手动实现

~~~java
/**
 * @author Deolin 2018/12/17
 */
public class MultimapCollectors<T, K, V> implements Collector<T, Multimap<K, V>, Multimap<K, V>> {

    private final Function<T, K> keyGetter;
    private final Function<T, V> valueGetter;

    public MultimapCollectors(Function<T, K> keyGetter, Function<T, V> valueGetter) {
        this.keyGetter = keyGetter;
        this.valueGetter = valueGetter;
    }

    public static <T, K, V> MultimapCollectors<T, K, V> toMultimap(Function<T, K> keyGetter,
            Function<T, V> valueGetter) {
        return new MultimapCollector<>(keyGetter, valueGetter);
    }

    public static <T, K, V> MultimapCollectors<T, K, T> toMultimap(Function<T, K> keyGetter) {
        return new MultimapCollector<>(keyGetter, v -> v);
    }

    @Override
    public Supplier<Multimap<K, V>> supplier() {
        return ArrayListMultimap::create;
    }

    @Override
    public BiConsumer<Multimap<K, V>, T> accumulator() {
        return (map, element) -> map.put(keyGetter.apply(element), valueGetter.apply(element));
    }

    @Override
    public BinaryOperator<Multimap<K, V>> combiner() {
        return (map1, map2) -> {
            map1.putAll(map2);
            return map1;
        };
    }

    @Override
    public Function<Multimap<K, V>, Multimap<K, V>> finisher() {
        return map -> map;
    }

    @Override
    public Set<Characteristics> characteristics() {
        return ImmutableSet.of(Characteristics.IDENTITY_FINISH);
    }

}
~~~






















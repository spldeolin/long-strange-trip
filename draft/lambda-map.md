---
title: 常用lambda（Map相关）

date: 2018-05-27 09:25:00

updated: 2018-05-27 09:25:00

tags:
- lambda

categories: Java

permalink: lambda-list
---



~~~java
package com.spldeolin.beginningmind.core.dto;

import java.util.Set;
import java.util.function.BiConsumer;
import java.util.function.BinaryOperator;
import java.util.function.Function;
import java.util.function.Supplier;
import java.util.stream.Collector;
import com.google.common.collect.ArrayListMultimap;
import com.google.common.collect.ImmutableSet;
import com.google.common.collect.Multimap;
import lombok.extern.log4j.Log4j2;

/**
 * @author Deolin 2018/12/17
 */
@Log4j2
public class MultimapCollector<T, K, V> implements Collector<T, Multimap<K, V>, Multimap<K, V>> {

    private final Function<T, K> keyGetter;
    private final Function<T, V> valueGetter;

    public MultimapCollector(Function<T, K> keyGetter, Function<T, V> valueGetter) {
        this.keyGetter = keyGetter;
        this.valueGetter = valueGetter;
    }

    public static <T, K, V> MultimapCollector<T, K, V> toMultimap(Function<T, K> keyGetter,
            Function<T, V> valueGetter) {
        return new MultimapCollector<>(keyGetter, valueGetter);
    }

    public static <T, K, V> MultimapCollector<T, K, T> toMultimap(Function<T, K> keyGetter) {
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





代码示例的注释用的都是“伪JSON”，仅仅是为了说明数据结构

将List映射到Map

// users [{id=3L, name=c, key=value1}, {id=2L, name=b, key=value2}, {id=1L, name=a, key=value3}]

Map<Long, User> usersById = users.stream().collect(Collectors.toMap(User::getId, one -> one));

// usersById {1={id=1, name=a, key=value3}, 2={id=2, name=b, key=value2}, 3={id=3, name=c, key=value1}}


其中，Collectors.toMap方法的第一个参数是作为Map的key来映射User对象的，key必须唯一，否则会倒是快速失败，抛出`java.lang.IllegalStateException: Duplicate key`异常


将List分组映射

可以看到，Collectors.toMap要求key必须唯一，所以他的返回值是Map<?, User>。
如果需要用非唯一的字段来映射，那可以换一个Collector

// users [{id=3L, name=same, key=value1}, {id=2L, name=same, key=value2}, {id=1L, name=diff, key=value3}]

Map<String, List<User>> userGroupsByName = users.stream().collect(Collectors.groupingBy(User::getName));

// userGroupsByName {same=[{id=3, name=same, key=value1}, {id=2, name=same, key=value2}], diff=[{id=1, name=diff, key=value3}]}

`Map<String, List<User>>`是一种很不优雅的返回类型，有的选的话，Deolin会使用`Multimap<String, User>`的方式来代替它
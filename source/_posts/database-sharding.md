---
title: sharding jdbc
date: 2020-04-30 23:22:17
tags:
    - 数据库
    - sharding jdbc
categories:
    - 数据库
---
### sharding jdbc

#### 分片策略
##### StandardShardingStrategy
标准分片策略。提供对SQL语句中的=, IN和BETWEEN AND的分片操作支持。StandardShardingStrategy只支持单分片键，提供PreciseShardingAlgorithm和RangeShardingAlgorithm两个分片算法。PreciseShardingAlgorithm是必选的，用于处理=和IN的分片；RangeShardingAlgorithm是可选的，用于处理BETWEEN AND分片，如果不配置RangeShardingAlgorithm，SQL中的BETWEEN AND将按照全库路由处理。
```xml
i_order:
  actual-data-nodes: ds$->{0..11}.i_order
  database-strategy:
    standard:
      shardingColumn: input_time
      preciseAlgorithmClassName: com.i.lubov.algorithm.DatabaseMonthDividedPrecise
      rangeAlgorithmClassName: com.i.lubov.algorithm.DatabaseMonthDividedRange
```
```java
public class DatabaseMonthDividedPrecise implements PreciseShardingAlgorithm<Date> {

    /**
     * 按input_time所属月份选择数据库
     *
     * @param availableTargetNames 数据库
     * @param shardingValue 分片键值
     * @return database
     */
    @Override
    public String doSharding(Collection<String> availableTargetNames, PreciseShardingValue<Date> shardingValue) {
        int month = LocalDate.fromDateFields(shardingValue.getValue()).getMonthOfYear();
        return availableTargetNames.toArray(new String[]{})[(month - 1)];
    }
}
```
##### ComplexShardingStrategy
复合分片策略。提供对SQL语句中的=, IN和BETWEEN AND的分片操作支持。ComplexShardingStrategy支持多分片键，由于多分片键之间的关系复杂，因此Sharding-JDBC并未做过多的封装，而是直接将分片键值组合以及分片操作符交于算法接口，完全由应用开发者实现，提供最大的灵活度。
```xml
i_order_info:
  actual-data-nodes: ds$->{0..15}.i_order_info_$->{0..15}
  database-strategy:
    complex:
      shardingColumns: order_no
      algorithmClassName: com.i.lubov.algorithm.DatabaseComplexKeys
  table-strategy:
    complex:
      shardingColumns: order_no
      algorithmClassName: com.i.lubov.algorithm.TableComplexKeysTable
```
```java
public class DatabaseComplexKeys implements ComplexKeysShardingAlgorithm<String> {

    private static final int MAX_SHARDING = 16;

    /**
     * 按单号后两位抹除16选择数据库
     *
     * @param availableTargetNames 数据库
     * @param shardingValue 分片键值
     * @return database
     */
    @Override
    public Collection<String> doSharding(Collection<String> availableTargetNames,
                                         ComplexKeysShardingValue<String> shardingValue) {
        Map<String, Collection<String>> shardingMap = shardingValue.getColumnNameAndShardingValuesMap();
        Set<String> sharding = new HashSet<>(MAX_SHARDING);
        shardingMap.forEach((k, v) -> v.forEach(no -> {
            if (StrUtil.isNotBlank(no) && no.length() > 4) {
                int datasourceIndex = Integer.parseInt(no.substring(no.length() - 2));
                sharding.add(availableTargetNames.toArray(new String[]{})[(datasourceIndex % MAX_SHARDING)]);
            }
        }));
        return sharding;
    }
}
```
##### InlineShardingStrategy
Inline表达式分片策略。使用Groovy的Inline表达式，提供对SQL语句中的=和IN的分片操作支持。InlineShardingStrategy只支持单分片键，对于简单的分片算法，可以通过简单的配置使用，从而避免繁琐的Java代码开发，如: ds$->{0..15}.i_waybill 表示i_waybill表按照user_id按16取模分成16个库，库名称为ds_0到ds_15。
```xml
i_waybill:
  actual-data-nodes: ds$->{0..15}.i_waybill
  database-strategy:
    inline:
      algorithm-expression: ds$->{user_id % 16}
      sharding-column: user_id
```
##### HintShardingStrategy
通过Hint而非SQL解析的方式分片的策略。
##### NoneShardingStrategy
不分片的策略。
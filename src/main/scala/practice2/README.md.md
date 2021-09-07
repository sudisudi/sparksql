#### 1. 创建sql表

```sql
spark-sql> desc student;
name	string	NULL
age	int	NULL
score	double	NULL
Time taken: 0.064 seconds, Fetched 3 row(s)
```

#### 2. 第一条sql语句

```sql
select name from (select * from student where true and age < 30) a where true and a.age > 20;
```

```shell
=== Applying Rule org.apache.spark.sql.catalyst.optimizer.CollapseProject ===
 Project [name#50]                                                                                                                                                  Project [name#50]
!+- Project [name#50]                                                                                                                                               +- Filter ((true AND (age#51 < 30)) AND (age#51 > 20))
!   +- Filter ((true AND (age#51 < 30)) AND (age#51 > 20))                                                                                                             +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#50, age#51, score#52], Partition Cols: []]
!      +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#50, age#51, score#52], Partition Cols: []]   
           
2021-09-03 14:32:08,122 WARN rules.PlanChangeLogger: 
=== Applying Rule org.apache.spark.sql.catalyst.optimizer.BooleanSimplification ===
 Project [name#50]                                                                                                                                               Project [name#50]
!+- Filter ((true AND (age#51 < 30)) AND (age#51 > 20))                                                                                                          +- Filter ((age#51 < 30) AND (age#51 > 20))
    +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#50, age#51, score#52], Partition Cols: []]      +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#50, age#51, score#52], Partition Cols: []]
           
2021-09-03 14:32:08,126 WARN rules.PlanChangeLogger: 
=== Result of Batch Operator Optimization before Inferring Filters ===
 Project [name#50]                                                                                                                                                     Project [name#50]
!+- Filter (true AND (age#51 > 20))                                                                                                                                    +- Filter ((age#51 < 30) AND (age#51 > 20))
!   +- Project [name#50, age#51, score#52]                                                                                                                                +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#50, age#51, score#52], Partition Cols: []]
!      +- Filter (true AND (age#51 < 30))                                                                                                                              
!         +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#50, age#51, score#52], Partition Cols: []]   
          
2021-09-03 14:32:08,130 WARN rules.PlanChangeLogger: 
=== Applying Rule org.apache.spark.sql.catalyst.optimizer.InferFiltersFromConstraints ===
 Project [name#50]                                                                                                                                               Project [name#50]
!+- Filter ((age#51 < 30) AND (age#51 > 20))                                                                                                                     +- Filter (isnotnull(age#51) AND ((age#51 < 30) AND (age#51 > 20)))
    +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#50, age#51, score#52], Partition Cols: []]      +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#50, age#51, score#52], Partition Cols: []]
           
2021-09-03 14:32:08,131 WARN rules.PlanChangeLogger: 
=== Result of Batch Infer Filters ===
 Project [name#50]                                                                                                                                               Project [name#50]
!+- Filter ((age#51 < 30) AND (age#51 > 20))                                                                                                                     +- Filter (isnotnull(age#51) AND ((age#51 < 30) AND (age#51 > 20)))
    +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#50, age#51, score#52], Partition Cols: []]      +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#50, age#51, score#52], Partition Cols: []]

```

可以看到，CollapseProject和BooleanSimplification被显示地apply出来，而CombineFilters在Infer Filters优化批次中隐式的做完了。

#### 3. 第二条sql语句

```sql
select distinct name, age a from (select name, age from student where 1 = '1') where age < 30 except (select name, age from student where age > 15) order by a;
```

```shell
=== Applying Rule org.apache.spark.sql.catalyst.optimizer.ReplaceExceptWithAntiJoin ===
 Sort [a#10 ASC NULLS FIRST], true                                                                                                                                              Sort [a#10 ASC NULLS FIRST], true
!+- Except false                                                                                                                                                                +- Distinct
!   :- Distinct                                                                                                                                                                    +- Join LeftAnti, ((name#11 <=> name#14) AND (a#10 <=> age#15))
!   :  +- Project [name#11, age#12 AS a#10]                                                                                                                                           :- Distinct
!   :     +- Filter (age#12 < 30)                                                                                                                                                     :  +- Project [name#11, age#12 AS a#10]
!   :        +- Project [name#11, age#12]                                                                                                                                             :     +- Filter (age#12 < 30)
!   :           +- Filter (1 = cast(1 as int))                                                                                                                                        :        +- Project [name#11, age#12]
!   :              +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#11, age#12, score#13], Partition Cols: []]         :           +- Filter (1 = cast(1 as int))
!   +- Project [name#14, age#15]                                                                                                                                                      :              +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#11, age#12, score#13], Partition Cols: []]
!      +- Filter (age#15 > 15)                                                                                                                                                        +- Project [name#14, age#15]
!         +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#14, age#15, score#16], Partition Cols: []]                     +- Filter (age#15 > 15)
                         +- Project [name#14, age#15]
!         +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#14, age#15, score#16], Partition Cols: []]                     +- Filter (age#15 > 15)
!                                                                                                                                                                                           +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#14, age#15, score#16], Partition Cols: []]

[WARN      ]116370  org.apache.spark.internal.Logging.logWarning(Logging.scala:69) 2021-09-03 15:38:42.977
=== Applying Rule org.apache.spark.sql.catalyst.optimizer.ReplaceDistinctWithAggregate ===
 Sort [a#10 ASC NULLS FIRST], true                                                                                                                                                 Sort [a#10 ASC NULLS FIRST], true
!+- Distinct                                                                                                                                                                       +- Aggregate [name#11, a#10], [name#11, a#10]
    +- Join LeftAnti, ((name#11 <=> name#14) AND (a#10 <=> age#15))                                                                                                                   +- Join LeftAnti, ((name#11 <=> name#14) AND (a#10 <=> age#15))
!      :- Distinct                                                                                                                                                                       :- Aggregate [name#11, a#10], [name#11, a#10]
       :  +- Project [name#11, age#12 AS a#10]                                                                                                                                           :  +- Project [name#11, age#12 AS a#10]
       :     +- Filter (age#12 < 30)                                                                                                                                                     :     +- Filter (age#12 < 30)
       :        +- Project [name#11, age#12]                                                                                                                                             :        +- Project [name#11, age#12]
       :           +- Filter (1 = cast(1 as int))                                                                                                                                        :           +- Filter (1 = cast(1 as int))
       :              +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#11, age#12, score#13], Partition Cols: []]         :              +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#11, age#12, score#13], Partition Cols: []]
       +- Project [name#14, age#15]                                                                                                                                                      +- Project [name#14, age#15]
+- Filter (age#15 > 15)                                                                                                                                                           +- Filter (age#15 > 15)
             +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#14, age#15, score#16], Partition Cols: []]                        +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#14, age#15, score#16], Partition Cols: []]

[WARN      ]116375  org.apache.spark.internal.Logging.logWarning(Logging.scala:69) 2021-09-03 15:38:42.982
=== Applying Rule org.apache.spark.sql.catalyst.optimizer.PushDownPredicates ===
 Sort [a#10 ASC NULLS FIRST], true                                                                                                                                                 Sort [a#10 ASC NULLS FIRST], true
 +- Aggregate [name#11, a#10], [name#11, a#10]                                                                                                                                     +- Aggregate [name#11, a#10], [name#11, a#10]
    +- Join LeftAnti, ((name#11 <=> name#14) AND (a#10 <=> age#15))                                                                                                                   +- Join LeftAnti, ((name#11 <=> name#14) AND (a#10 <=> age#15))
       :- Aggregate [name#11, a#10], [name#11, a#10]                                                                                                                                     :- Aggregate [name#11, a#10], [name#11, a#10]
       :  +- Project [name#11, age#12 AS a#10]                                                                                                                                           :  +- Project [name#11, age#12 AS a#10]
!      :     +- Filter (age#12 < 30)                                                                                                                                                     :     +- Project [name#11, age#12]
!      :        +- Project [name#11, age#12]                                                                                                                                             :        +- Filter ((1 = cast(1 as int)) AND (age#12 < 30))
!      :           +- Filter (1 = cast(1 as int))                                                                                                                                        :           +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#11, age#12, score#13], Partition Cols: []]
!      :              +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#11, age#12, score#13], Partition Cols: []]         +- Project [name#14, age#15]
!      +- Project [name#14, age#15]                                                                                                                                                         +- Filter (age#15 > 15)
!         +- Filter (age#15 > 15)                                                                                                                                                              +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#14, age#15, score#16], Partition Cols: []]
!            +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#14, age#15, score#16], Partition Cols: []]

[WARN      ]116398  org.apache.spark.internal.Logging.logWarning(Logging.scala:69) 2021-09-03 15:38:43.005
=== Applying Rule org.apache.spark.sql.catalyst.optimizer.ConstantFolding ===
 Sort [a#10 ASC NULLS FIRST], true                                                                                                                                              Sort [a#10 ASC NULLS FIRST], true
 +- Aggregate [name#11, a#10], [name#11, a#10]                                                                                                                                  +- Aggregate [name#11, a#10], [name#11, a#10]
    +- Aggregate [name#11, a#10], [name#11, a#10]                                                                                                                                  +- Aggregate [name#11, a#10], [name#11, a#10]
       +- Project [name#11, age#12 AS a#10]                                                                                                                                           +- Project [name#11, age#12 AS a#10]
          +- Join LeftAnti, ((name#11 <=> name#14) AND (age#12 <=> age#15))                                                                                                              +- Join LeftAnti, ((name#11 <=> name#14) AND (age#12 <=> age#15))
             :- Project [name#11, age#12]                                                                                                                                                   :- Project [name#11, age#12]
!            :  +- Filter ((1 = cast(1 as int)) AND (age#12 < 30))                                                                                                                          :  +- Filter (true AND (age#12 < 30))
             :     +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#11, age#12, score#13], Partition Cols: []]               :     +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#11, age#12, score#13], Partition Cols: []]
             +- Project [name#14, age#15]                                                                                                                                                   +- Project [name#14, age#15]
                +- Filter (age#15 > 15)                                                                                                                                                        +- Filter (age#15 > 15)
                   +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#14, age#15, score#16], Partition Cols: []]                     +- HiveTableRelation [`default`.`student`, org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, Data Cols: [name#14, age#15, score#16], Partition Cols: []]

[WARN      ]116432  org.apache.spark.internal.Logging.logWarning(Logging.scala:69) 2021-09-03 15:38:43.039

```

只出来了4条优化规则，没有FoldablePropagation。可能前面别名被处理掉了导致破坏了FoldablePropagation的场景条件。
1. #### 在SqlBase.g4文件中添加语法| SHOW VERSION #showVersion，以及关键词VERSION，使用antlr4进行编译

2. #### 在SparkSqlParser类中添加visit方法

   ```scala
   /*
   override def visitShowVersion(ctx: ShowVersionContext): LogicalPlan = 		                                                                         withOrigin(ctx) {
       ShowVersionCommand()
   }
   */
   ```

3. #### 在command包中添加case class ShowVersionCommand

   ```scala
   /*
    * Licensed to the Apache Software Foundation (ASF) under one or more
    * contributor license agreements.  See the NOTICE file distributed with
    * this work for additional information regarding copyright ownership.
    * The ASF licenses this file to You under the Apache License, Version 2.0
    * (the "License"); you may not use this file except in compliance with
    * the License.  You may obtain a copy of the License at
    *
    *    http://www.apache.org/licenses/LICENSE-2.0
    *
    * Unless required by applicable law or agreed to in writing, software
    * distributed under the License is distributed on an "AS IS" BASIS,
    * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    * See the License for the specific language governing permissions and
    * limitations under the License.
    */
   
   package org.apache.spark.sql.execution.command
   
   import org.apache.spark.sql.{Row, SparkSession}
   import org.apache.spark.sql.catalyst.expressions.{Attribute, AttributeReference}
   import org.apache.spark.sql.types.StringType
   
   case class ShowVersionCommand() extends RunnableCommand {
   
     override val output: Seq[Attribute] =
       Seq(AttributeReference("version", StringType, nullable = true)())
   
     override def run(sparkSession: SparkSession): Seq[Row] = {
       /** 如何更精确获取spark的版本, 这里还没有想出来*/  
       val sparkVersion = System.getenv("SPARK_HOME")
       val javaVersion = System.getProperty("java.version")
       val outputString = Tuple2(sparkVersion, javaVersion).toString()
       Seq(Row(outputString))
     }
   }
   ```

   4. #### 对spark源码进行编译和打包

      ```shell
      C:\Users\86136\Desktop\spark-3.1.2>sbt package -Phive -Phive-thriftserver
      [info] welcome to sbt 1.4.6 (Oracle Corporation Java 1.8.0_161)
      [info] loading settings for project spark-3-1-2-build from plugins.sbt ...
      [info] loading project definition from C:\Users\86136\Desktop\spark-3.1.2\project
      
        | => spark-3-1-2-build / update 2621s
        
      [error] java.lang.RuntimeException: Failing because of negative scalastyle result
      java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
      [error]         at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
      [error]         at java.lang.Thread.run(Thread.java:748)
      [error] (sql / scalaStyleOnCompile) Failing because of negative scalastyle result
      [error] Total time: 435 s (07:15), completed 2021-9-2 17:35:15
      ```

   


## 作业一：为 Spark SQL 添加一条自定义命令

SHOW VERSION；
显示当前 Spark 版本和 Java 版本。
步骤：
通过修改 SqlBase.g4 文件添加自定义命令，需要在原生 SqlBase.g4 文件中添加下面 4 处
1)
statement
    : query    
    | SHOW VERSION                                 #showVersion  
2)
nonReserved
//--DEFAULT-NON-RESERVED-START
    : ADD
    | VERSION
3) 
ansiNonReserved
//--ANSI-NON-RESERVED-START
    : ADD
    | VERSION
4)
//============================
// Start of the keywords list
//============================
//--SPARK-KEYWORD-LIST-START
VERSION: 'VERSION';

执行 spark-catalyst maven module 的 anltr4:anltr4 goal
修改 SparkSqlParser.scala 文件, 重写 visitShowVersion 方法
  override def visitShowVersion(ctx: ShowVersionContext): LogicalPlan = withOrigin(ctx) {
    ShowVersionCommand()
  }
在 commands.scala 类中增加实现类 ShowVersionCommand，代码如下，
case class ShowVersionCommand() extends RunnableCommand {
  override def run(sparkSession: SparkSession): Seq[Row] = {
    Seq(Row(util.Properties.versionNumberString, System.getProperty("java.version")))
  }
}
在 spark 源码根目录下执行命令"./build/sbt package -DskipTests -Phive -Phive-thriftserver"
将 SPARK_HOME 环境变量设置成 spark 源码根目录，然后执行./bin/spark-sql，进入 spark-sql 控制台之后，执行 show version 命令，过程如下图，
<img width="635" alt="image" src="https://user-images.githubusercontent.com/33747692/167296049-4d25041f-6852-463c-aac6-02c9c97c20cf.png">


## 作业二：构建 SQL 满足如下要求

通过 set spark.sql.planChangeLog.level=WARN，查看：

构建一条 SQL，同时 apply 下面三条优化规则：
CombineFilters
CollapseProject
BooleanSimplification
构建一条 SQL，同时 apply 下面五条优化规则：
ConstantFolding
PushDownPredicates
ReplaceDistinctWithAggregate
ReplaceExceptWithAntiJoin
FoldablePropagation

解析：
第一条 sql 如下，spark plan 日志见文件SparkSqlChangeLog。
语句：
select a.address 
from (
	select name, address,age 
	from customers 
	where 1="1" and age > 5
) a 
where a.age<30

第二条 sql 如下，spark plan 日志见文件。
语句：
(select a.address , a.age + (100 + 80) , Now() z 
from (
	select distinct name, age , address  
	from customers
) a 
where a.age>10 order by z)  
except 
(select a.address , a.age + (100 + 80), Now() z 
from (
	select distinct name, age , address  
	from customers
) a 
where a.name="saya");

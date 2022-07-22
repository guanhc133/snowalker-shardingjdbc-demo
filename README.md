# snowalker-shardingjdbc-demo

> shardingjdbc3.x 整合，实现自定义主键生成，自定义分库分表策略，可扩展，可读性更好


| 项目关联文章列表 |
|  :------ |
| [我说分布式之分库分表](http://wuwenliang.net/2019/03/11/%E6%88%91%E8%AF%B4%E5%88%86%E5%B8%83%E5%BC%8F%E4%B9%8B%E5%88%86%E5%BA%93%E5%88%86%E8%A1%A8/) |
| [跟我学shardingjdbc之shardingjdbc入门](http://wuwenliang.net/2019/03/12/%E8%B7%9F%E6%88%91%E5%AD%A6shardingjdbc%E4%B9%8Bshardingjdbc%E5%85%A5%E9%97%A8/) |
| [跟我学shardingjdbc之使用jasypt加密数据库连接密码](http://wuwenliang.net/2019/03/14/%E8%B7%9F%E6%88%91%E5%AD%A6shardingjdbc%E4%B9%8B%E4%BD%BF%E7%94%A8jasypt%E5%8A%A0%E5%AF%86%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%9E%E6%8E%A5%E5%AF%86%E7%A0%81/) |
| [跟我学shardingjdbc之分布式主键及其自定义](http://wuwenliang.net/2019/03/25/%E8%B7%9F%E6%88%91%E5%AD%A6shardingjdbc%E4%B9%8B%E5%88%86%E5%B8%83%E5%BC%8F%E4%B8%BB%E9%94%AE%E5%8F%8A%E5%85%B6%E8%87%AA%E5%AE%9A%E4%B9%89/#qrcode) |
| [跟我学shardingjdbc之自定义分库分表策略-复合分片算法自定义实现](http://wuwenliang.net/2019/03/26/%E8%B7%9F%E6%88%91%E5%AD%A6shardingjdbc%E4%B9%8B%E8%87%AA%E5%AE%9A%E4%B9%89%E5%88%86%E5%BA%93%E5%88%86%E8%A1%A8%E7%AD%96%E7%95%A5-%E5%A4%8D%E5%90%88%E5%88%86%E7%89%87%E7%AE%97%E6%B3%95%E8%87%AA%E5%AE%9A%E4%B9%89%E5%AE%9E%E7%8E%B0/#qrcode) |

个人心得：

com.snowalker.shardingjdbc.snowalker.demo.complex.sharding.util.StringUtil<br />
> 取ASICC码是为了数值运算，因为传进来的主键不是数字<br />
> 通过该ASCII码对库取商，对表取余，得到库表下标<br />
> 对库--获得id的ASICC码值，<br />
>  1,值和表的数量取余，余数肯定在表的数量范围内，<br />
 > 2,eg:总共16个表（4库16表，每个库分别4个用户表），任意数字和16取余，其范围必定在0-15范围（所有库中表的范围也在0-15）<br />
>  3,以上获取的余值和库的数（4）取商，因为余值范围0-15，所以商的范围为0-3 （库的范围是 ds00,ds01.....ds03）<br />
 > 所以无论怎么计算都不会超过库表数量的边界<br />

> 对表--获得id的ASICC码<br />
>  同1,2<br />
>  3,以上获取的余值和每个库中的表的数量（总表数量16除以总库数量4=4）取余，因为余值范围0-15，所以取余的范围为0-3 （一个库中表的范围是 table00,table00.....table03）<br />



> com.snowalker.shardingjdbc.snowalker.demo.complex.sharding.strategy.SnoWalkerComplexShardingDB<br />

> sharding-jdbc查找数据大致流程<br />
> 1,application-db-config.properties配置需要分片表的分片键，指定自定义实现的分片算法algorithmClassName<br />
> 2,执行sql前判断sql中是否存在分片键，存在则走自定义分片算法的doSharding方法（不存在就查找全库全表），入参为availableTargetNames前面配置所有的库名，shardingValues本次sql里涉及的><br /> > 分片键字段名，字段值和字段所在的表<br />
> 前面我们自定义的分片键值的实现（UD+db+table+01+yyMMddHHmmssSSS+机器id+序列号id），可以很清楚知道这是UD用户表，且对应的库表号是多少，<br />
 > 拿到库就可以匹配我们在配置文件中配置的对应数据源，接着在查询语句中拼接表号（比如：user01表）查询对应的表数据<br />

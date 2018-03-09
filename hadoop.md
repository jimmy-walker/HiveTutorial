# 命令格式
hadoop fs：使用面最广，可以操作任何文件系统。
hadoop dfs与hdfs dfs：只能操作HDFS文件系统相关（包括与Local FS间的操作），前者已经Deprecated，一般使用后者。

# 具体命令
## shell命令（这个方法是最基本的）
```linux
hadoop fs  -ls	<path>	表示对hdfs下一级目录的查看 #常用
hadoop fs -lsr	<path>	表示对hdfs目录的递归查看
hadoop fs -mkdir	<path>	创建目录 #常用
hadoop fs  -put	<src>	<des>	从linux上传文件到hdfs #常用
hadoop fs  -get	<src>	<des>	从hdfs下载文件到linux
hadoop fs  -text	<path>	查看文件内容
hadoop fs  -rm	<path>	表示删除文件
hadoop fs  -rmr	<path>	表示递归删除文件
```
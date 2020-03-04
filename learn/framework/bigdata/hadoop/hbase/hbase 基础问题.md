#   habase 基础问题

##   为什么不建议在 HBase 中使用过多的列族 
*	HBase 中每张表的列族个数建议设在1~3之间
*	列族数对 Flush 的影响
	*	一个列簇 对应一个 memstore , 每个memstore 会刷新hfile
	*	
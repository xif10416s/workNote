#  kylin汇总.md
*	https://mp.weixin.qq.com/s?__biz=MzU5OTQ1MDEzMA==&mid=2247487357&idx=3&sn=247e866342fd79b5248cde41a9083229&chksm=feb5f800c9c2711692efd1c6cb9d5988a983eb8933f7dec4422828fd86c9d298b2ba8c7dbfd0&scene=126&sessionid=1583196171&key=82eb7d7c3126bb97bd7f8252a5645c61a279b8b0eb7192239ad9018201f11b52e1ec00f108e0d7e7dd4026d399ae075f337b956bd5bdea2f9a77471597724dbc8ce57e62ad074dfa1b2eae33c93c8582&ascene=1&uin=Mjk1NTAwNzcwMg%3D%3D&devicetype=Windows+10&version=62080079&lang=zh_CN&exportkey=AdQG9XKTEaOxrsibEoh3W3E%3D&pass_ticket=lP8OsTR37xbFEWnIwa2Xn1VX8Vh7OwxsRyAj8EK0TxrSqVbSWMTmkYI7CXFz9GjF
*	docker image/ cdh 安装
	*	https://mp.weixin.qq.com/s?__biz=MzAwODE3ODU5MA==&mid=2653079740&idx=1&sn=79736d0fcf3f6aa9c6250f8acddef22b&chksm=80a4b64db7d33f5b30dd120d41b364c5fc24a2c7b6901055a7d0fae26a8514c1eabd788bf542&scene=126&sessionid=1585528474&key=e9cba451e717dd4975fac99cae22be1e8b15d030e5a27752561d61235d7c705cee9c691525714e46ab999920c33018072073decd44b45b5be2c67fc8230a9106e2c9dda403d692c89a5c82043a856b1f&ascene=1&uin=Mjk1NTAwNzcwMg%3D%3D&devicetype=Windows+10&version=62080079&lang=zh_CN&exportkey=AQNB%2B07p96KsIZZaBbM%2FpuM%3D&pass_ticket=ZwhehbJCtGxyjucif4FQ6gC5LoGO%2FPPF9MSBgIdXDriaqijxpYQjHWDd2IesNpDZ
	*	docker pull apachekylin/apache-kylin-standalone:3.0.1
```
docker run -d \
-m 8G \
-p 7070:7070 \
-p 8088:8088 \
-p 50070:50070 \
-p 8032:8032 \
-p 8042:8042 \
-p 16010:16010 \
apachekylin/apache-kylin-standalone:3.0.1
```
*	管理页面 --  docker exec -it  465bd44809c2  /bin/sh
	* 	http://192.168.99.100:7070/kylin/login
	* 	Kylin 页面：http://127.0.0.1:7070/kylin/
		*	Kylin默认账号为:ADMIN默认密码:KYLIN
	*    Hdfs NameNode 页面：http://127.0.0.1:50070
	*    Yarn ResourceManager 页面：http://127.0.0.1:8088
	*    HBase 页面：http://127.0.0.1:60010

#### demo
*	sample cube，可以参考http://kylin.apache.org/cn/docs/tutorial/kylin_sample.html。
*	对于 kylin_streaming_cube，需要设置 KAFKA_HOME，然后执行 ${KYLIN_HOME}/bin/sample-streaming.sh，该脚本会在 localhost:9092 broker 中创建名为 kylin_streaming_topic 的 Kafka Topic，它也会每秒随机发送 100 条 messages 到 kylin_streaming_topic，然后你可以对 kylin_streaming_cube 进行构建。

#### demo解释
*	https://blog.csdn.net/shenshouniu/article/details/83543560
#安全接口
## https + 双向认证 + 签名

## client 调用问题：
*  http://blog.csdn.net/a351945755/article/details/22782229
*  javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
    -   没有指定trustStore，需要把服务端证书添加到可信证书路径
    -   下载指定网址的证书，http://blog.csdn.net/faye0412/article/details/6883879


##https
*   http://blog.csdn.net/zyw_anquan/article/details/7829053
*   证书，其实就是一对公钥和私钥
*   流程
    -   客户端访问服务端，服务端返回证书（公钥）
    -   客户端验证证书，使用公钥+生成随机值（对称加密值）生成加密内容返回服务端
    -   服务端通过私钥解密内容，获取客户端的随机值，用这个值对称加密解密
*   双向认证
    -   客户端访问服务端，服务端返回证书
    -   客户端获取证书并验证
    -   客户端发送自己的证书到服务端，服务端验证客户端证书
    -   客户端用服务端的公钥+随机码生成通话密钥发送给服务端
    -   服务端用私钥获取通话密钥，对称加密通信
*   SSL防止的是中间人攻击跟网络监听
 

##签名
*   apikey/apisecretkey这种模式确实是做的ACL，访问权限控制用的。
    -   都是服务端生成提供的
    -   apikey , 验证是否是我们服务注册登记过的用户
    -   apisecretkey，一端密文，用于产生请求签名，与应用绑定
*   流程
    -   服务端生成apikey 和 apisecretkey 给客户端 ，并保存
    -   客户端通过apisecretkey加密生成签名，与key 和请求内容一起发给服务端
    -   服务端接受请求，通过key 找到对应的apisecretkey，同样方式加密生成签名，与客户端签名比对
    -   签名一致返回

##认证
*   timestamp（时间戳） 与 nonce（随机数），解决Replay-Attack问题
    -   都是防止中间攻击的安全机制
    -   timestamp（时间戳）,服务端对请求时间戳进行判断，如超过5分钟，认定为重放攻击，请求无效。
    -   nonce（随机数）,持久化唯一请求


   
##重放攻击（Replay attack)
*   是一种网络攻击，它恶意的欺诈性的重复或拖延正常的数据传输。
*   假设Alice向Bob认证自己。Bob要求她提供密码作为身份信息。同时，Eve窃听两人的通讯，并记录密码。在Alice和Bob完成通讯后，Eve联系Bob，假装自己为Alice，当Bob要求密码时，Eve将Alice的密码发出，Bob认可和自己通讯的人是Alice。
*   https本身就是防止重放攻击的    


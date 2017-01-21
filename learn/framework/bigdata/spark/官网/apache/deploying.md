#Deploying
##Submitting Applications
###发布模式
*   client mode -- 适合client 和 cluster 靠近，网络传输快
                     
    client的submit的时候启动并运行driver程序，与worker和master集群交互，适合spark shell

*   cluster mode-- 适合client 和 spark 集群很远
    
    为了减少driver和executors的网络开销，driver由worker启动，在worker上运行

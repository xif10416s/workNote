##作业提交与dagschedule操作
[参考](http://blog.jasonding.top/tags/Spark/)

![](http://7xo3b7.com1.z0.glb.clouddn.com/00000000000000000000000000000000_pt_b_photo_012.jpg)

![](http://7xo3b7.com1.z0.glb.clouddn.com/00000000000000000000000000000000_pt_b_photo_031.jpg)

##DADScheduler.handleJobSubmit
![](http://7xo3b7.com1.z0.glb.clouddn.com/00000000000000000000000000000000_pt_b_photo_015.jpg)

##DAGSchedual#submitMissingTask

stage如何生成taskset
如果一个stage的所有parent stage已经计算完成或者存在于cache中就会调用这个方法来提交该stage包含的tasks

![](http://7xo3b7.com1.z0.glb.clouddn.com/00000000000000000000000000000000_pt_b_photo_027.jpg)

##driver ，master， worker 通信关系
![](http://7xo3b7.com1.z0.glb.clouddn.com/00000000000000000000000000000000_pt_b_photo_025.jpg)





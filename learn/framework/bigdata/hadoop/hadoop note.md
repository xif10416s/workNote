# hadoop
## block size
*   Block size is just an indication to HDFS how to split up and distribute the files across the cluster - there is not a physically reserved number of blocks in HDFS (you can change the block size for each individual file if you wish)
*   A final note - having lots to small files means that your name node will require more memory to track them (blocks sizes, locations etc), and its generally less efficient to process 128x1MB files than single 128MB file (although that depends on how you're processing it)
### http://blog.csdn.net/clerk0324/article/details/50888016
### https://www.cnblogs.com/Dhouse/p/6901028.html?utm_source=itdadao&utm_medium=referral

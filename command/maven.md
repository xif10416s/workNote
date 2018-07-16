maven
mvn dependency:sources -DdownloadSources=true 
mvn dependency:sources -DincludeGroupIds=org.slf4j -Dclassifier=sources -DdownloadSources=true
无法导入source:
-Xmx1024m -DsocksProxyHost=127.0.0.1 -DsocksProxyPort=1086
导入 cdh 依赖
https://blog.csdn.net/u010453283/article/details/78266520

      <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>2.6.0-cdh5.9.1</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-core</artifactId>
            <version>2.6.0-cdh5.9.1</version>
        </dependency>



        <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-hdfs -->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
            <version>2.6.0-cdh5.9.1</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hive.hcatalog</groupId>
            <artifactId>hive-hcatalog-streaming</artifactId>
            <version>${hive.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hive.hcatalog</groupId>
            <artifactId>hive-hcatalog-core</artifactId>
            <version>${hive.version}</version>
        </dependency>


    </dependencies>

    <repositories>

        <repository>

            <id>cloudera</id>

            <url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>

        </repository>
    </repositories>
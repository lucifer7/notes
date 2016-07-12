#### Integrate with Maven
`<!-- apache hadoop start -->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>${hadoop-version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-core</artifactId>
            <version>${hadoop-version}</version>
        </dependency>
        <!-- apache hadoop end -->
        `
version: 2.7.2

#### For windows
download: https://github.com/srccodes/hadoop-common-2.2.0-bin
Setup HADOOP_HOME, and add to path

# Maven使用记录

1. 跳过测试两种方式

   1. 不执行测试用例，但编译测试用例类生成相应的class文件至target/test-classes下：

      ```shell
      mvn package -DskipTests
      ```

   2. 不执行测试用例，也不编译测试用例类：

      ```shell
      mvn package -Dmaven.test.skip=true
      ```

      
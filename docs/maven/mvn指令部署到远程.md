# Maven

> <a src="[Maven – 将第三方 JAR 部署到远程存储库的指南 (apache.org)](https://maven.apache.org/guides/mini/guide-3rd-party-jars-remote.html)">Maven官方文档</a>



**将本地jar打包成pom推送到远程厂库**

```bash

### install 官方指令
mvn install:install-file -Dfile=<path-to-file> -DgroupId=<group-id> -DartifactId=<artifact-id> -Dversion=<version> -Dpackaging=<packaging>

### 实际操作
mvn install:install-file -Dfile=F:\code\libs\test-java-sdk-1.0.0.jar -DgroupId=com.igsshan.api -DartifactId=test-java-sdk -Dversion=1.0.0 -Dpackaging=jar


### deploy 官方指令
mvn deploy:deploy-file -DgroupId=<group-id> \
  -DartifactId=<artifact-id> \
  -Dversion=<version> \
  -Dpackaging=<type-of-packaging> \
  -Dfile=<path-to-file> \
  -DrepositoryId=<id-to-map-on-server-section-of-settings.xml> \
  -Durl=<url-of-the-repository-to-deploy>
  
### 实际操作
mvn deploy:deploy-file -Dmaven.test.skip=true -DgroupId=com.igsshan.api -DartifactId=test-java-sdk -Dversion=1.0.0-SNAPSHOT -Dpackaging=jar -Durl=http://'username':'password'@192.168.211.92:8081/repository/maven-snapshots/ -Dpackaging=jar -Dfile=D:\igsshan\test-java-sdk-1.0.0.jar -DpomFile=D:\igsshan\test-java-sdk-1.0.0.pom -DrepositoryId=maven-snapshots

# 由于install文件路径太长;最好是将生产的jar和pom文件拷贝出来;放到一个干净的目录下以便使用

# 参数说明：
# -Dfile       jar包文件所在目录
# -DgroupId      分组
# -DartifactId    名称
# -Dversion      版本
# -DrepositoryId  服务器的表示ID，在 nexus 的 configuration 可以看到
# -Durl        私服上仓库的位置，打开 nexus——>repositories 菜单，可以看到该路径
# -DpomFile      pom.xml所在目录【非必填】
```



**`也可以图形化操作`**

**贴图**

<div>
    <image width="100%" src="images/maven/maven上传本地jar包01.png">
    <br/>
    <image width="100%" src="images/maven/maven上传本地jar包02.png">
    <br/>
    <image width="100%" src="images/maven/maven上传本地jar包03.png">
</div>


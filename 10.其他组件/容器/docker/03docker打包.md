本栏介绍如果在本地环境，避开Jenkins进行打包，优点是可以跳过代码评审，在开发环境对开发内容进行测试，而且不依赖Jenkins

# 根据jar包在服务器上生成镜像（推荐）

- 根据本地项目maven生成jar包

- 在jar包移到服务器上，在对应目录下新建一个dockerfile文件

  ![](https://youcai922.github.io/99.src/img/docker利用jar包打包.png)

  ```dockerfile
  #依赖的jdk版本，如果没有则会启动去拉取该镜像
  FROM openjdk:8-jdk-alpine
  COPY ./*.jar /app.jar
  #端口
  EXPOSE 5010
  ENTRYPOINT ["java","-Djava.security.egd=file:/dev./urandom","-jar","/app.jar"]
  ```

- 运行以下命令将jar包打包为镜像（后面有一个.）

  docker build -f dockerfile -t oilfiled-task:v1.1 .



# 利用idea远程连接服务器docker生成镜像（不推荐）

- 服务器安装docker
- 更改docker配置，允许其他主机访问该docker服务
  - vim /usr/lib/systemd/system/docker.service
  - 找到该文件的ExecStart一行，在后缀加上-H tcp://0.0.0.0:2375

![](https://youcai922.github.io/99.src/img/docker开放远程连接端口.png)

```
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -H tcp://0.0.0.0:2375
```

- 本地idea下载docker插件（一般不用下载）

![](https://youcai922.github.io/99.src/img/ideadocker插件下载.png)

- 配置docker连接

![](https://youcai922.github.io/99.src/img/idea配置远程docker连接.png)

- 更改项目pom文件配置

  ```
  <plugin>
      <groupId>com.spotify</groupId>
      <artifactId>docker-maven-plugin</artifactId>
      <version>1.0.0</version>
      <!--将插件绑定在package这个phase上，用户只需执行mvn package，就会自动执行mvn docker:build-->
      <executions>
          <execution>
              <id>build-image</id>
              <phase>package</phase>
              <goals>
                  <goal>build</goal>
              </goals>
              <configuration>
                  <dockerHost>http://192.168.73.133:2375</dockerHost>
                  <dockerDirectory>${project.basedir}</dockerDirectory>
                  <!--项目名称-->
                  <imageName>${docker.registry}/${project.artifactId}:${tag}</imageName>
                  <buildArgs>
                      <WAR_FILE>${project.build.finalName}</WAR_FILE>
                  </buildArgs>
                  <resources>
                      <resource>
                          <targetPath>/</targetPath>
                          <directory>${project.build.directory}</directory>
                          <include>${project.build.finalName}.war</include>
                      </resource>
                  </resources>
              </configuration>
          </execution>
      </executions>
      <configuration>
          <buildArgs>
              <artifactId>${project.artifactId}</artifactId>
              <VERSION>${project.version}</VERSION>
          </buildArgs>
      </configuration>
  </plugin>
  ```

- 编写项目dockerfile

  ```
  #基础运行环境，没有会自动下载
  FROM openjdk:8-jdk-alpine
  COPY ./target/*.jar /app/app.jar
  #端口
  EXPOSE 5010
  ENTRYPOINT ["java","-Djava.security.egd=file:/dev./urandom","-jar","/app.jar"]
  ```

  ![](https://youcai922.github.io/99.src/img/项目dockerfile.png)

- 利用maven进行打包

​	![](https://youcai922.github.io/99.src/img/利用maven进行打包.png)

- 验证打包是否成功

​	![](https://youcai922.github.io/99.src/img/服务器验证是否打包成功.png)



## 也可以在idea里面操作远程的镜像运行

![](https://youcai922.github.io/99.src/img/idea远程操作镜像运行.png)

![](https://youcai922.github.io/99.src/img/idea远程镜像配置.png)
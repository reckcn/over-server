## 一 Dockerfile简介

Dockerfile类似脚本，用于制作docker镜像，可以让容器实现自动化。  

Dockerfile的操作步骤：
- 1、找一个镜像： `ubuntu`
- 2、创建一个容器： `docker run ubuntu`
- 3、进入容器： `docker exec -it 容器 命令`
- 4、操作： `各种应用配置....`
- 5、构造新镜像： `docker commit`

Dockerfile使用规范：
- 1、大： 首字母必须大写D
- 2、空： 尽量将Dockerfile放在空目录中。
- 3、单： 每个容器尽量只有一个功能。
- 4、少： 执行的命令越少越好。

Dockerfile使用命令：
```
# 构建镜像命令格式：
docker build -t [镜像名]:[版本号][Dockerfile所在目录]

# 构建样例：
docker build -t nginx:v0.2 /opt/dockerfile/nginx/

# 参数详解：
-t 指定构建后的镜像信息
/opt/dockerfile/nginx/ 则代表Dockerfile存放位置，如果是当前目录则用 . 表示
```

## 二 Dockerfile实战

### 2.1 构建一个web服务

需求：定义一个dockerfile，发布一个go的web服务。  

假设现在root目录下有一个Dockerfile，以及一个`hello.go`的源码文件：
```
# 定义父镜像
FROM go:1.13

# 定义作者信息
MAINTAINER ruyue ruyuejun@gmail.com

# 添加go文件到容器中
ADD /root/hello.go app.go

# 定义容器执行的命令
CMD go run app.go
```

利用Dockerfile构建一个镜像，并指定镜像名称与构建位置
```
docker build -t myweb:1.0 .
```

### 2.2 构建一个jdk1.8镜像

使用Dockerfile快速基于Centos创建一个定制化jdk1.8镜像：
```
# 创建Dockerfile专用目录
mkdir /home/docker/images/java -p
cd docker/images/java/
vim Dockerfile                          # 此名称固定
```

Dockerfile内容：
```
# 基础镜像
FROM centos:7

# 镜像作者
MAINTAINER ruyue ruyuejun@gmail.com

# 工作目录
WORKDIR /usr

# 在容器中执行命令：创建java的安装目录
RUN mkdir /usr/local/java

# 将宿主机当前目录的jdk文件上传到容器中的安装目录
ADD jdk-8u1710linux-x64.tar.gz /usr/local/java/

# 配置环境
ENV JAVA_HOME /usr/local/java/jdk1.8.0_171
ENV JRE_HOME $JAVA_HOME/jre
```

利用Dockerfile构建一个镜像，并指定镜像名称与构建位置
```
docker build -t myjdk:1.8 .
```

## 三 常用命令

RUN：表示当前镜像构建时候运行的命令，如果有确认输入的话，一定要在命令中添加 -y，如果命令较长，那么可以在命令结尾使用 \ 来换行，生产中，推荐使用面数组的格式。
```
# shell模式，类似于 /bin/bash -c command
RUN <command>   
RUN echo hello

# exec 模式，类似于 RUN["/bin/bash", "-c", "command"]                  
RUN["executable", "param1", "param2"] 
RUN["echo", "hello"]
```

CMD：指定容器启动时默认执行的命令,每个Dockerfile只能有一条CMD命令，如果指定了多条，只有最后一条会被执行,如果在启动容器的时候使用docker run 指定的运行命令，那么会覆盖CMD命令。
```
#格式：
CMD ["executable","param1","param2"] (exec 模式)推荐
CMD command param1 param2 (shell模式)
CMD ["param1","param2"] 提供给ENTRYPOINT的默认参数；

# 示例
CMD ["/usr/sbin/nginx","-g","daemon off；"]
```

ENTRYPOINT：和CMD 类似都是配置容器启动后执行的命令，并且不会被docker run 提供的参数覆盖，每个Dockerfile 中只能有一个ENTRYPOINT，当指定多个时，只有最后一个起效。生产中我们可以同时使用ENTRYPOINT 和CMD，想要在docker run 时被覆盖，可以使用"docker run --entrypoint"
```
#格式：
ENTRYPOINT ["executable", "param1","param2"] (exec 模式)
ENTRYPOINT command param1 param2 (shell 模式)
```

ADD：将指定的`<src> `文件复制到容器文件系统中的`<dest>`，src 指的是宿主机，dest 指的是容器。所有拷贝到container 中的文件和文件夹权限为0755,uid 和gid 为0，如果文件是可识别的压缩格式，则docker 会帮忙解压缩。
```
# 格式：
ADD <src>... <dest>
ADD ["<src>",... "<dest>"]
```

注意：
- 1、如果源路径是个文件，且目标路径是以/ 结尾， 则docker 会把目标路径当作一个目录，会把源文件拷贝到该目录下;如果目标路径不存在，则会自动创建目标路径。
- 如果源路径是个文件，且目标路径是不是以/ 结尾，则docker 会把目标路径当作一个文件。如果目标路径不存在，会以目标路径为名创建一个文件，内容同源文件；如果目标文件是个存在的文件，会用源文件覆盖它，当然只是内容覆盖，文件名还是目标文件名。如果目标文件实际是个存在的目录，则会源文件拷贝到该目录下。注意，这种情况下，最好显示的以/ 结尾，以避免混淆。
- 如果源路径是个目录，且目标路径不存在，则docker 会自动以目标路径创建一个目录，把源路径目录下的文件拷贝进来。如果目标路径是个已经存在的目录，则docker 会把源路径目录下的文件拷贝到该目录下。
- 如果源文件是个压缩文件，则docker 会自动帮解压到指定的容器目录中

COPY:COPY 指令和ADD 指令功能和使用方式类似。只是COPY 指令不会做自动解压工作。单纯复制文件场景，Docker 推荐使用COPY。
```
#格式：
COPY <src>... <dest>
COPY ["<src>",... "<dest>"]
```

VOLUME：VOLUME 指令可以在镜像中创建挂载点，这样只要通过该镜像创建的容器都有了挂载点，通过VOLUME 指令创建的挂载点，无法指定主机上对应的目录，是自动生成的
```
#格式：
VOLUME ["/data"]
```

EVN：设置环境变量，可以在RUN 之前使用，然后RUN 命令时调用，容器启动时这些环境变量都会被指定
```
#格式：
ENV <key> <value> （一次设置一个环节变量）
ENV <key>=<value> ... （一次设置一个或多个环节变量）
```

WORKDIR：切换目录，为后续的RUN、CMD、ENTRYPOINT 指令配置工作目录。相当于cd.也可以使用多个WORKDIR 指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。例如
```
#格式：
WORKDIR /path/to/workdir (shell 模式)
```

USER与ARG：指定运行容器时的用户名和UID，后续的RUN 指令也会使用这里指定的用户。如果不输入任何信息，表示默认使用root 用户
```
#格式：
USER daemon
ARG <name>[=<default value>]
```

注意：ARG 指定了一个变量在docker build 的时候使用，可以使用`--build-arg <varname>=<value>`来指定参数的值，不过如果构建的时候不指定就会报错。  

ONBUILD：触发器指令，当一个镜像A被作为其他镜像B的基础镜像时，这个触发器才会被执行，新镜像B在构建的时候，会插入触发器中的指令。使用场景对于版本控制和方便传输，适用于其他用户。
```
# 示例
ONBUILD COPY ["index.html","/var/www/html/"]
```

## 四 一些贴士

### 4.1 镜像尽量小

对于Docker镜像来说，不应该使用过大的体积，越大越慢，也更容器出现问题。 

### 4.2 清理构建冗余

Dockerfile中多个RUN指令推荐使用&&连接符以及反斜杠换行做法将其包含在一个RUN指令。  

RUN指令在构建时，也会拉取一些构建工具，这些工具也会残留在镜像中移交到生产环境，这是不合适的，需要在构建完成后进行清理。  

一般采用过阶段构建的方式，多阶段构建包含多个FROM指令，每个FROM都是一个新的构建阶段，并且可以方便的复制之前阶段的构建：
```dockerfile
FROM node:latest AS client
WORKDIR /usr/src/app/views
COPY react-app .
RUN npm install
RUN npm run build

FROM maven:latest AS server
WORKDIR /usr/src/app/
COPY pom.xml .
RUN mvn -B -f pom.xml -s /usr/share/maven/ref/settings-docker.xml dependency
\:resolve
COPY . .
RUN mvn -B -s /usr/share/maven/ref/settings-docker.xml package -DskipTests

FROM java:8-jdk-alpine AS app
RUN adduser -Dh /home/user1 user1
WORKDIR /work
COPY --from=client /usr/src/app/views/build/ .
WORKDIR /app
COPY --from=server /usr/src/target/test.jar .
ENTRYPOINT ["java", "-jar", "/app/test.jar"]
CMD ["--spring.profiles.active=postgres"]
```

Dockerfile中有3个FROM指令，每个FROM构成一个单独的构建阶段，各个阶段都会在内部从0开始编号：
- 阶段0，示例中叫做client，拉取了node镜像，使用RUN显著增加了镜像大小
- 阶段1，示例中叫做server，拉取了maven镜像，也产生了构建工具
- 阶段2，示例中叫做app，拉取jdk镜像，但是使用`COPY from`后，只会拉取client和server中的应用代码

### 4.3 镜像缓存

第一次构建很慢，之后的构建都会很快，这是docker构建镜像时采用了缓存机制，第一次构建时会拉取镜像，并会将该构建内容（镜像层）缓存下来，为后续的构建提供复用。

注意：
- 一旦有指令在缓存中未命中，则后续的整个构建过程都不再使用缓存！！  
- 如果copy的内容发生变化，也不会使用缓存（docker内部会就算每个复制文件的Checksum值）

取消缓存：
```
docker build --nocache=true
```

### 4.4 合并镜像

当镜像中层数太多时，合并是很好的优化方式，但是这种合并也带来了弊端：无法共享镜像层，导致存储空间的低效利用，push和pull的镜像体积也会更大。  

合并操作：
```
docker image build --squash
```
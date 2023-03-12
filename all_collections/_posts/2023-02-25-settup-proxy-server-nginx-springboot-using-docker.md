---
layout: post
title: Cài đặt một Nginx Proxy Server cơ bản cho Java Spring Boot app sử dụng Docker
date: 2023-01-25
categories: ["nginx", "container", "dev", "vi"]
---

Trong mạng máy tính, proxy server dựa trên giao thức TCP được coi như là một _trạm trung gian_ đứng giữa máy chủ (host machine) và người dùng (end-user). Khi người dùng gửi request đến máy chủ, thay vì để request được gửi thẳng trực tiếp đến máy chủ, proxy server sẽ tiếp nhận những request đó, xử lí (tùy vào business logic), rồi sau đó mới chuyển tiếp qua cho máy chủ. Quá trình tiếp nhận request từ người dùng tới proxy server còn được gọi là _downstream flow_, **downstream ám chỉ vị trí của người dùng đến proxy** còn quá trình chuyển tiếp request từ proxy server tới máy chủ được gọi là _upstream flow_, **upstream ám chỉ vị trí từ proxy đến máy chủ**.

<!-- ![proxy server](https://user-images.githubusercontent.com/123849429/222787964-3f8af036-d616-40bd-b6b6-b4fec9e4aa43.png) -->

![proxy server](https://user-images.githubusercontent.com/123849429/222877015-f8914c68-dc0f-4576-a2b8-8c9b68678d60.png)

Khi nhắc đến việc cài đặt proxy server, có rất nhiều lựa chọn để ta có thể dùng để thực hiện, trong đó Nginx được coi như là một trong những sự lựa chọn tối ưu nhất vì [nhiều lí do](https://www.glator.com/blog/reasons-to-choose-nginx/#:~:text=Another%20main%20reason%20why%20you%20should%20be%20thinking,an%20exceptional%20job%20at%20each%20one%20of%20them.), ví dụ như việc dễ cài đặt, dễ dùng, tài liệu tham khảo đầy đủ, tối ưu về hiệu năng, cộng đồng người khá đông, và miễn phí, etc.

Trong bài viết lần này, chúng ta sẽ cùng nhau làm:

- Cài đặt và tạo Docker image cho Spring Boot app
- Cài đặt và tạo Docker image cho Nginx
- Cài đặt và tạo Docker network để Spring Boot và Nginx containers có thể làm việc chung
- Cài đặt proxy server cho Spring Boot app sử dụng Nginx

## Prerequisites

- Docker hoặc Docker desktop
- POSIX OS (Linux, Unix)
  - WSL hoặc WSL2
- Spring Boot v2.0+
- JDK 11+
- Maven 3+

## Getting Started

Trong bài viết này, mình sẽ không đi sâu vào phần code của Spring Boot. Thay vào đó, để băt đầu chúng ta sẽ sử dụng một project có sẵn. Bạn có thể clone project thông qua git URL dưới đây.

```
$ mkdir ~/nginx-springboot-webserver
$ git clone -b nginx-springboot git@github.com:frankiie12a9/spring-boot-nginx-simple-reverse-proxy.git
```

Sau khi clone project vể máy, ta sẽ chuyển hướng tới root của project, nơi có `pom.xml` file.

```
$ cd ./spring-boot-nginx-simple-reverse-proxy
```

Tiếp theo, chúng ta cần clean và reset lại project (xóa bản target folder được built trước đó), rồi sau đó build để tạo JAR file.

```
$ mvn clean package
```

Tiếp đến là run project để xem nó có chạy ngon lành ở local hay không.

```
$ mvn spring-boot:run
```

### Spring Boot and Docker

Giờ chúng ta cần tạo một Docker image cho Spring Boot project. Để có thể thực hiện được, trước tiên, chúng ta sẽ tạo một `Dockerfile` file tại root của project, sau đó copy đoạn code dưới đây rồi paste vào trong file nhé.

```Dockerfile
FROM eclipse-temurin:17-jdk-alpine
VOLUME /tmp

ARG DB_SOURCE_PW
ARG DB_SOURCE_URL
ARG DB_SOURCE_USERNAME
ARG JAR_FILE

ENV SPRING_DATASOURCE_PASSWORD=${DB_SOURCE_PW}
ENV SPRING_DATASOURCE_URL=${DB_SOURCE_URL}
ENV SPRING_DATASOURCE_USERNAME=${DB_SOURCE_USERNAME}

COPY ${JAR_FILE} /tdd-0.0.1-SNAPSHOT.jar

ENTRYPOINT ["java","-jar","/tdd-0.0.1-SNAPSHOT.jar"]
```

Giải thích:

- `FROM`: cái thẻ này là [base image](https://docs.docker.com/glossary/#base-image) hay còn gọi là build source, luôn luôn đứng ở vị trí trước tiên trong Dockerfile. Nó là đại diện cho môi trường của image. Nói một cách dễ hiểu hơn thì, nếu project sử dụng Java Spring Boot thì môi trường sẽ phải là Java, JavaScript thì môi trường sẽ là NodeJS, C# thì sẽ là .NET, etc. `eclipse-temurin:17-jdk-alpine` chính là Java JDK mà chúng ta sẽ dùng trong project lần này.
- `VOLUME`: thẻ này dùng để lưu trữ các loại data bên ngoài container's file system. Khi image được tạo ra, chúng ta sẽ sử dụng cái image đó để run container (E.g., Spring Boot app container), và mỗi container đó khi được sử dụng sẽ có cho mình những data riêng biệt, ví dụ như app database, access logs, error logs, etc.
  Docker container sử dụng volume bảo quản những loại đata đó mà không bị phụ thuộc vào OS hay máy chủ.
- `ARG`: thẻ này dùng để chỉ định biến môi trường cho image trong quá trình build. Mỗi lần image đc build, tùy thuộc vào độ phức tạp trong việc cài đặt image, chúng có thể sẽ có cho mình những biến môi trường cần để có thể được build. Ví dụ, như trong project lần này chúng ta có sử dụng H2 Database (là một loại [in-memory database](https://en.wikipedia.org/wiki/In-memory_database#:~:text=An%20in-memory%20database%20%28%20IMDB%2C%20also%20main%20memory,management%20systems%20that%20employ%20a%20disk%20storage%20mechanism.)), để có thể tích hợp database, cơ bản, chúng ta cần `DATASOURCE_PASSWORD`, `DATASOURCE_URL`, và `DATASOURCE_USER`, đó cũng được coi là những biến môi trường.
- `ENV`: thẻ này chức năng cũng tương tự như `ARG`, tuy nhiên khác ở hỗ nó phải được khai báo và gán giá trị tĩnh(statically) tại Dockerfile, còn `ARG` thì sẽ được gán giá trị động (dynamically) lúc build image.
- `COPY`: thẻ này sẽ thực hiện chức năng sao chép files/folders từ source file (`JAR_FILE`) qua destination file (`./tdd-0.0.1-SNAPSHOT.jar`) ở bên trong image trong lúc build.
- `ENTRYPOINT`: thẻ này dùng để cài đặt cấu hình thực thi mệnh lệnh cho container. Ví dụ, để thực thi một Java project, cơ bản ta cần Java (`java`), jar hoặc war files (`jar`), và project.jar hoặc project.war file (`/tdd-0.0.1-SNAPSHOT.jar`). E.g., _java -jar /tdd-0.0.1-SNAPSHOT.jar_

Oke, Sau khi Dockerfile được cài đặt xong, chúng ta sẽ build nó bằng câu lệnh dưới đây.

```
$ docker build -t <tên_image>:<tag_của_image>  \
    --build-arg JAR_FILE=./target/tdd-0.0.1-SNAPSHOT.jar \
    --build-arg DB_SOURCE_URL=jdbc:h2:mem:<your_database> \
    --build-arg DB_SOURCE_USERNAME=sa \
    --build-arg DB_SOURCE_PW= \
    .
```

Giải thích:

- `-t`: thẻ này dùng để biểu thị tên và version hoặc tag mà chúng ta muốn gán cho image.
- `--build-arg`: thẻ này thực dùng để biểu thị image được build sẽ bao gồm các biến môi trường đã được khởi tạo và định nghĩa ở Dockerfile (có 4 biến tường ứng với 4 thẻ ở đây).

Sau khi thực thi câu lệnh bên trên, một image tên `<tên_image>:<tag_của_image>` sẽ được tạo. Bạn có thể kiểm tra danh sách images hiện có thông qua:

```
$ docker images
```

Tuy nhiên, kiểm tra danh sách những images không thôi là chưa đủ. Chúng ta cần phải check xem `Volume`, các biến môi trường `ARG`, `ENV`, và `ENTRYPOINT` được gán ở Dockerfile đã đúng và đẩy đủ hay chưa.

```
$ docker image inspect <tên_image>:<tag_của_image>
```

Khi câu lệnh được thực thi, bạn sẽ thấy rất nhiều giá trị được lưu dưới dạng JSON, và trong đó sẽ có những trường như `RepoTags`, `Config`, và `Entrypoint`. Nếu giá trị của chúng khớp với những gì chúng ta gán ở trên (Dockerfile) thì là oke rồi đó nha.

```
 "Id": "sha256:8a334dd1xxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "RepoTags": [
    "<your_image>:<your_image_tag>"
  ],
 "Config": {
     ...,
    "Env": [
      "...",
      "SPRING_DATASOURCE_URL=jdbc:h2:mem:testdb",
      "SPRING_DATASOURCE_USERNAME=sa",
      "SPRING_DATASOURCE_PASSWORD="
    ],
  }
  "...",
  "Volumes": {
    "/tmp": {}
  },
  "Entrypoint": [
    "java",
    "-jar",
    "/tdd-0.0.1-SNAPSHOT.jar"
  ],
```

Sau khi image đã được tạo, giờ là lúc chúng ta dùng nó để run Docker container cho Spring Boot app bằng cách thực thi câu lệnh sau đây.

```
$  docker run --name <tên_container> -p <host_port>:<container_port> -d -t <tên_image>:<tag_của_image>
```

Giải thích:

- `-d`: hay đầy đủ là `--detach`, thẻ này ám chỉ container sẽ được chạy trong chế độ _background_ (tức là process sẽ được run mà ta không nhìn thấy nó hoạt động, hay không tương tác trực tiếp thông qua giao diện, e.g., `Logging`, `System monitoring` , trái ngược với _foreground_ là process là ta sẽ nhìn thấy và có thể tương tác trực tiếp, e.g., `VSCode`, `Browser`)

Oke, xong rồi. Giờ chúng ta cần test xem container đã được run thành công hay không.

```
$ docker logs <your_container_name> -f
```

Nếu không có vấn đề gì thì bạn sẽ thấy được output của container đang chạy giống như dưới đây.

```
 .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.7.7)

2023-03-03 18:25:00.721  INFO 1 --- [           main] com.mvc.tdd.TddApplication               : Starting TddApplication v0.0.1-SNAPSHOT using Java 17.0.6 on b202a5e2d9c5 with PID 1 (/tdd-0.0.1-SNAPSHOT.jar started by root in /)
2023-03-03 18:25:00.725  INFO 1 --- [           main] com.mvc.tdd.TddApplication               : No active profile set, falling back to 1 default profile: "default"
2023-03-03 18:25:01.635  INFO 1 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data JPA repositories in DEFAULT mode.
2023-03-03 18:25:01.701  INFO 1 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 53 ms. Found 3 JPA repository interfaces.
2023-03-03 18:25:02.357  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 1500 (http)
2023-03-03 18:25:02.371  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2023-03-03 18:25:02.372  INFO 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.70]
2023-03-03 18:25:02.453  INFO 1 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2023-03-03 18:25:02.453  INFO 1 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1650 ms
2023-03-03 18:25:02.484  INFO 1 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2023-03-03 18:25:02.719  INFO 1 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2023-03-03 18:25:02.729  INFO 1 --- [           main] o.s.b.a.h2.H2ConsoleAutoConfiguration    : H2 console available at '/h2-console'. Database available at 'jdbc:h2:mem:testdb'
2023-03-03 18:25:02.873  INFO 1 --- [           main] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [name: default]
2023-03-03 18:25:02.929  INFO 1 --- [           main] org.hibernate.Version                    : HHH000412: Hibernate ORM core version 5.6.14.Final
2023-03-03 18:25:03.118  INFO 1 --- [           main] o.hibernate.annotations.common.Version   : HCANN000001: Hibernate Commons Annotations {5.1.2.Final}
2023-03-03 18:25:03.234  INFO 1 --- [           main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.H2Dialect
Hibernate: drop table if exists math_grade CASCADE
Hibernate: drop table if exists science_grade CASCADE
Hibernate: drop table if exists student CASCADE
Hibernate: create table math_grade (id integer generated by default as identity, grade double, student_id integer, primary key (id))
Hibernate: create table science_grade (id integer generated by default as identity, grade double, student_id integer, primary key (id))
Hibernate: create table student (id integer generated by default as identity, email_address varchar(255), first_name varchar(255), last_name varchar(255), primary key (id))
2023-03-03 18:25:03.914  INFO 1 --- [           main] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
2023-03-03 18:25:03.922  INFO 1 --- [           main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
2023-03-03 18:25:04.316  WARN 1 --- [           main] JpaBaseConfiguration$JpaWebConfiguration : spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
2023-03-03 18:25:04.441  INFO 1 --- [           main] o.s.b.a.w.s.WelcomePageHandlerMapping    : Adding welcome page template: index
2023-03-03 18:25:04.641  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 1500 (http) with context path ''
2023-03-03 18:25:04.649  INFO 1 --- [           main] com.mvc.tdd.TddApplication               : Started TddApplication in 4.563 seconds (JVM running for 5.521)
2023-03-03 18:28:59.181  INFO 1 --- [nio-1500-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2023-03-03 18:28:59.181  INFO 1 --- [nio-1500-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2023-03-03 18:28:59.185  INFO 1 --- [nio-1500-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 4 ms
```

Hoặc bạn cũng có thể mở browser lên và truy cập thử vào `localhost:<container_port>` để kiểm tra kết quả nha.

![](https://user-images.githubusercontent.com/123849429/222810603-8c2b9d7c-54a3-4419-90cb-58ef0567a989.png)

### Nginx and Docker

Roài, tiếp đến chúng ta sẽ làm việc với Nginx. *Phần này đơn gian và dễ hơn phần trên nên nhanh lắm nha*😊. Để tiết kiệm thời gian, với Nginx thì chúng ta sẽ không cần cài đặt Dockerfile mà chỉ cần pull cái [official image](https://hub.docker.com/_/nginx/) của nó từ Docker Hub về cho nhanh nha.

```
$ docker pull nginx
```

Sau khi pull xong, ta chạy container.

```
$ docker run --name <tên_container> -p 80:80 -d nginx
```

Kiểm tra xem container đã được chạy thành công hay chưa.

```
$ docker ps | grep "<tên_container>"
```

### Docker Networking

Sau khi đã cài đặt và chạy xong Spring Boot và Nginx containers. Chúng ta cần phải tạo một network cho chúng có thể giao tiếp và làm việc được với nhau. Để làm được điều này, chúng ta cần đến [Docker network](https://docs.docker.com/network/).

> Để tiết kiệm thời gian nên mình sẽ không đi sâu và giải thích về cơ chết hoạt động của Docker networking. Bạn có thể tham khảo trên Docker networking manuals nha.

Oke, đầu tiên chúng ta cần tạo một network như sau.

```
$ docker network create <tên_network>
```

Sau đó chúng ta sẽ kế nối network vừa tạo với hai containers Spring Boot và Nginx đã chạy ở trên.

```
$ docker network connect <tên_network> <spring-boot_container>
$ docker network connect <tên_network> <nginx_container>
```

Giải thích:

- `connect`: cái option này sẽ giúp Docker kết nối network `<tên_network>` với container `<container>`.
- Bạn có thể thực thi `docker network create --help` và `docker network connect --help` để xem thêm cách dùng của nó.

Sau khi thực thi hai lệnh trên, ta cần kiểm tra xem network `<tên_network>` đã kết nối thành công với Spring Boot và Nginx containers hay chưa.

```
$ docker network inspect <tên_network>
```

Sau khi dòng lệnh trên được thực thi, chúng ta sẽ thấy thông tin của cái network `<tên_network>`, trong đó sẽ có một trường tên `Containers`, bạn xem bên trong nó có xuất hiện hai containers Spring Boot và Nginx hay không nha. Nếu có thì oke đó nha.

```
{
  "...",
  "Containers": {
     "0ad9b9663c2e768ab90b8b04b735efbd85fc9268d163375cd3908d32a2c74f60": {
     "Name": "<tên_nginx_container>",
     "EndpointID": "d5ab93d5fef250793fd9cffdd1bd5a781be5aa69d1351053266c7b889a2a825a",
     "MacAddress": "02:42:ac:12:00:04",
     "IPv4Address": "172.18.0.4/16",
     "IPv6Address": ""
    },
    "b202a5e2d9c5502df4043ac4572547f05410328996caeb822286f3584831cbec": {
      "Name": "<tên_springboot_container>",
       "EndpointID": "aa5273e6bc37efb4672680c832659543533afd15fe4eba12ccf3b9e3a2d9f3a1",
       "MacAddress": "02:42:ac:12:00:03",
       "IPv4Address": "172.18.0.3/16",
       "IPv6Address": ""
      }
  },
}
```

### Proxy Server

Oke, giờ là bước cuối này... Để có thể biến Nginx hoạt động như một proxy server với Spring Boot, trước tiên, chúng ta cần phải cài đặt Nginx sao cho nó có thể tiếp nhận HTTP request (incoming request) và chuyển tiếp (forward) đến máy chủ (host server, tức backend của Spring Boot). Rồi, giờ ta sẽ vào bên trong Nginx container đang chạy để thao tác.

```
$ docker exec -it <tên_nginx_container> /bin/bash
```

Chuyển hướng về cái file `etc/nginx/nginx.conf` và dùng vim để chỉnh sửa nội dung của file (nếu bạn không dùng vim thì cũng có thể dùng `nano`).

```
root@0ad9b9663c2e:/# vim etc/nginx/nginx.conf
```

> _Lưu ý_ Nếu trong lúc thực thi bằng `vim`, nếu bạn gặp lỗi này:
>
> ```
> bash: vim: command not found
> ```

> Thì chỉ cần tải nó về là giải quyết được nha.
>
> ```
> apt-get update && apt-get install vim
> ```

Copy đoạn code dưới đây rồi paste vào file `nginx.conf` (kiểm tra xem bên trong file còn nội dung hay không, nếu có thì xóa đi rồi paste vào) nha.

```bash
# địng nghĩa user cho nginx instance hiện tại
user nginx

# chỉ đinh tham số "auto" cho worker process (let Nginx decide what and how to optimize itself!)
worker_process auto;

# set errors logging nếu xảy ra lỗi
error_log /var/log/nginx/error.log notice;

# chỉ định nơi chứa PID (PROCESS IDs)
pid /var/run/nginx.pid;

# for handling connections and events. `1024` là tham số events/connections
# mà instance hiện tại có thể handle và xử lí
events {
  worker_connections 1024;
}

# cái block này là quan trọng nhất.
# nó là nơi để tiếp nhận và xử lí HTTP request (incoming request) và forward (outgoing request) tới host server
http {
  include /etc/nginx/mime.types;
  default_type application/octet-stream;

  # đây chính là một `virtual server` của Nginx instance.
  # block `location` nó sẽ tiếp nhận HTTP request đc
  # gửi đến port 80 và sau đó nó sẽ chuyển tiếp đến `http://<tên_springboot_container>:<container_port>` như một proxy server
  server {
    listen 80;

    # store access logs
    access_log /var/log/nginx/access.log;
    # store error logs
    error_log /var/log/nginx/error.log;

    location / {
        proxy_pass http://<tên_springboot_container>:<container_port>;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
  }

  log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"  '

  sendfile on;

  keepalive_timeout 65;
}
```

Giải thích:

- Những phần quan trọng mình comment ở trên code rồi nha.

Rồi, sau khi copy cái đoạn code trên vào file ta lưu file và thoát ra ngoài bằng tổ hợp phím sau:
ấn `esc`, ấn tiếp `:`, rồi ấn tiếp `wq`, cuối cùng gõ `exit` (E.g., `root@0ad9b9663c2e:/# exit`).

Restart Nginx container để updates được áp dụng nha.

```
$ docker restart <tên_nginx_container>
```

### Try it out!

Sau khi xong xuôi các bước trên, tức là cũng xong rồi đó 😊. Giờ ta truy cập vào browser và test cái port của Nginx container `localhost:80`. Nếu kết quả của bạn cũng giống mình thì tức là giờ chúng ta đã cài đặt thành công một Nginx proxy server cho Spring Boot app rồi đó.

![ok-image](https://user-images.githubusercontent.com/123849429/222830592-3a1be9c1-5209-45d1-ac65-f4a88898fa18.png)

Ta bật Devtools (ấn F12) rồi kiểm tra xem request có được nhận bởi Nginx hay không.

![nginx-connection](https://user-images.githubusercontent.com/123849429/222831055-152ff0b1-8dea-4ecf-b764-7aaaad3414f1.png)

> _BONUS_, trong trường hợp bạn muốn check acess logs hay error logs của Nginx, thì thực hiện theo duới đây.

- Đi vào bên trong container đang chạy
  > $ docker exec -it <tên_nginx_container> /bin/bash
- kiểm tra access logs thông qua `tail`, `acesss.log`
  > root@0ad9b9663c2e:/# tail -f var/log/nginx/access.log
- Kiểm tra error logs
  > root@0ad9b9663c2e:/# tail -f var/log/nginx/error.log

## Conclusion

Trong bài viết lần này, chúng ta đã cùng nhau tạo images cho Java Spring Boot app và Nginx sau đó là chạy containers dựa trên những image đó, tiếp đến ta đã tạo docker networking để những containers đó có thể làm việc được cùng với nhau, và cuối cùng chúng ta đã cài đặt Nginx để nó có thể hoạt động như một proxy server cho Spring Boot app. Bài viết cũng khá dài, nếu bạn đọc được đến đây thì mình cảm ơn nha. Hi vọng bạn tìm được điều gì đó hữu ích trong bài viết này... Chúc các bạn học tập thành công 😊!

**_Mình viết blog để tổng hợp lại những gì mình đã học, và cũng như học cách trình bày sao cho người khác có thể hiểu được, nên bài viết có thể tồn tại bug hoặc chưa hoàn thiện đâu đó. Nếu có gì liên quan đến bài viết, cần giúp debug, etc. thì đừng ngần ngại mà hãy cứ nhắn tin cho mình qua [Facebook](https://www.facebook.com/frankiie12a9/) nha._**

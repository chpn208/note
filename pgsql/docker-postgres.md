### 1. docker 安装postgresql

1. docker pull postgres:9.4  安装9.4 版本

2. docker 启动 postgresql容器

   ```sh
    docker run --name my_postgres   #指定contaner name
    -v dv_pgdata:/var/lib/postgresql/data # 指定数据卷--映射容器目录到宿主机目录
    -e POSTGRES_PASSWORD=123456 # -e 环境变量， 指定postgresql 密码
    -p 5432:5432  # 端口映射
    -d postgres:9.4 # -d 后台运行
   
   ```

3. 进入容器终端

   ```sh
   docker exec #运行容器中的命令
   -it # -i :即使没有附加也保持STDIN 打开 -t 分配一个伪终端
   my_postgres # 指定容器名称
   /bin/bash # 指定打开的目录
   ```

4. 打开psql

   ```sh
   psql -U postgres -W   # 下一步输入密码 123456
   ```

5. 查看配置文件目录

   ```sql
   select name, setting from pg_settings where category='File Locations' ;
   ```

   

```
       name        |                 setting                  
-------------------+------------------------------------------
 config_file       | /var/lib/postgresql/data/postgresql.conf
 data_directory    | /var/lib/postgresql/data
 external_pid_file | 
 hba_file          | /var/lib/postgresql/data/pg_hba.conf
 ident_file        | /var/lib/postgresql/data/pg_ident.conf
```

### 2. 数据库管理

#### 2.1 配置文件

1. postgresql.conf

   该文件包含一些<b>能用</b>(基础)的设置，比如内存分配、新建database的默认存储位置、PostgreSQL服务器的IP地址、日志的位置以及许多其他设置

   可通过查询pg_settings视图查询当前设置

   ```sql
   select name,context,unit,setting,boot_val,reset_val from pg_settings where name in('listen_addresses','max_connections','shared_buffers','effective_cache_size','work_mem','maintenance_work_mem')
   order by context,name;
   ```

   ```sh
            name         |  context   | unit | setting | boot_val  | reset_val 
   ----------------------+------------+------+---------+-----------+-----------
    listen_addresses     | postmaster |      | *       | localhost | *
    max_connections      | postmaster |      | 100     | 100       | 100
    shared_buffers       | postmaster | 8kB  | 16384   | 1024      | 16384
    effective_cache_size | user       | 8kB  | 524288  | 524288    | 524288
    maintenance_work_mem | user       | kB   | 65536   | 65536     | 65536
    work_mem             | user       | kB   | 4096    | 4096      | 4096
   ```

   context : 如果该值为postmaster 修改该行后要重启PostgreSQL服务器才能生效；如果为user，那么只需执行一次重新加载可全局生效。（重启与重新加载的区别）

   unit : 表示该记录的单位

   setting :指当前设置，boot_val 指默认设置，reset_val 指重启服务器或重新加载设置之后的新设置

   

   +++

   

   2. pg_hba.conf

   指定了允许哪些用户以何种方式连接到PostgreSQL。针对该文件的修改可动态生效。

```sh
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     trust
# IPv4 local connections:
host    all             all             127.0.0.1/32            trust # 1
# IPv6 local connections:
host    all             all             ::1/128                 trust #
# Allow replication connections from localhost, by a user with the
# replication privilege.
#local   replication     postgres                                trust
#host    replication     postgres        127.0.0.1/32            trust
#host    replication     postgres        ::1/128                 trust
```

1. 身份验证模式，一般有ident、trust、md5以及password。**这些都是干啥的？**
   * trust 这是最不安全的 ，允许用户“自证清白”（不用密码就连接到数据库）。只要源端IP地址、连接用户名、要访问的database名都与该条规则切尔西，就可以连上来。
   *  md5 要求连接发起者携带用md5算法加密的密码
   * password 明文密码验证，不推荐
   * ident 系统会将请求发起者的操作系统用户映射为PostgreSQL数据库内部用户，并以该内部用户的权限登录，且无需提供密码---> 操作系统用户映射为数据库用户



3.  配置文件的重新加载

   ``` sh
   pg_ctl reload -D you_data_directory_here
   ```

   

#### 2.2 连接管理

可用于解决慢查询，锁库锁表等问题

1. 查出活动连接列表及其进程ID

   ```sql
   SELECT * FROM pg_stat_activity;
   ```

2. 取消连接上的活动查询

   ```sql
   SELECT pg_cancel_backend(procid);
   ```

3. 终止该连接

   ```sql
   SELECT pg_terminate_backend(procid);
   ```

   如果你未停止某个连接上正在执行的语句就直接终止该连接，那么这些语句此时也会被停止掉

#### 2.3 角色


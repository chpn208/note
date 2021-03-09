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

   


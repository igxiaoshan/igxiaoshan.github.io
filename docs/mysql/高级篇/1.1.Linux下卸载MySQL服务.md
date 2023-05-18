# MySQL服务卸载

## 查看是否安装过MySQL

- 如果是用rpm安装,检查一下 `rpm package`

  ```sh
  rpm -qa | grep -i mysql # -i 忽略大小写
  ```

- 检查mysql service

  ```sh
  systemctl status mysqld.service # 检查mysql服务状态
  ```

## MySQL卸载

- 关闭MySQL服务

  ```sh
  systemctl stop mysql.service
  ```

- 检查当前MySQL安装状况

  ```sh
  rpm -qa | grep -i mysql
  
  # 或
  
  yum list installed | grep mysql
  ```

- 卸载上述命令查询出的已安装的程序

  ```sh
  yum remove mysql--xxx
  # 务必卸载干净,反复执行 'npm -qa | grep -i mysql'确实是否有卸载残留 
  ```

- 卸载mysql相关文件

  - 查找相关文件

    ```sh
    find / -name mysql
    ```

  - 删除上述命令查询出的相关文件

    ```sh
    rm -rf xxx
    ```

- 删除 my.cnf

  ```sh
  # my.cnf 文件就是 windows下的 my.ini 配置文件
  # 存放在 /etc/my.cnf
  rm -rf /etc/my.cnf
  ```



至此卸载完成!!! 可以重新再次安装MySQL服务
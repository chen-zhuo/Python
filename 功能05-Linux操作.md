# Linux操作

## 文件下载

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import paramiko


def ssh_down(ip, port, username, password, server, local):
    """
    # 从服务器上下载文件
    :param ip: 服务器IP
    :param port: 端口
    :param username:用户
    :param password: 密码
    :return:
    """
    # 实例化一个trans对象# 实例化一个transport对象
    transport = paramiko.Transport((ip, port))
    # 建立连接
    transport.connect(username=username, password=password)
    # 实例化一个 sftp对象,指定连接的通道
    sftp = paramiko.SFTPClient.from_transport(transport)
    # 将历史数据压缩包下载到本地文件中
    sftp.get(server, local)
    transport.close()


if __name__ == '__main__':
    # 服务器文件路径
    server_route = '/home'
    # 服务器文件名称
    server_name = 'qualification_company_association.tar'
    # 本地存储路径
    local_route = 'D:/chenzhuo/longna_distributed_reboot/test'
    # 本地文件名称
    local_name = 'qualification_company_association.tar'
    # 下载文件
    ssh_down("服务器IP", 22, "登录的用户名", "连接密码", f'{server_route}/{server_name}',f'{local_route}/{local_name}')

```

## 代码部署

执行下面的代码前提是，公钥已经配置在了服务器上，能通过私钥连接服务器：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Time    : 2021/10/22
# @Author  : chenzhuo
# @Desc    : 代码部署
# !/usr/bin/env python
# -*- coding: utf-8 -*-

import time
import paramiko

class Deployment():
    def __init__(self):
        # 9台服务器
        self.linux_server = (
            '''服务器的IP地址'''
        )
        # 命令流（\n就是确认）
        self.command = (
            # 进入docker
            "docker attach data_server\n",
            # 暂停心跳（作用等于Ctrl+C）
            "\x03\n",
            # 进入项目文件夹
            "cd /home/py_crawl/longna_distributed_reboot/\n",
            # 启动虚拟环境
            "source /home/py_crawl/venvs/longna_distributed_reboot/bin/activate\n",
            # 扔掉本地分支修改
            "git checkout .\n",
            # 切换到master分支
            "git checkout master\n",
            # 删除dev分支
            "git branch -D dev\n",
            # 拉取线上dev分支
            "git fetch -f origin dev:dev\n",
            # 切换到dev分支
            "git checkout dev\n",
            # 关闭所有super服务
            "supervisorctl stop all",
            # 重新启动其他服务器的super服务
            "supervisorctl start celery_task:fill_task control_task:sys_run_reboot crontab_server:dispatch_task\n",
            # 重新启动215服务器的super服务
            "supervisorctl start celery_task:fill_token_pool celery_task:ln_beat crontab_server:transfer_to_sql global_server:pro_api celery_task:fill_task control_task:sys_run_reboot crontab_server:dispatch_task celery_task:fill_proxy_pool celery_task:fill_cookie_pool celery_task:fill_cookie_pool\n",
            # 启动心跳
            "python /home/py_crawl/longna_distributed_reboot/zkeeper/heart_server.py\n"
        )

    def sshexecmd(self, ip):
        try:
            ssh = paramiko.SSHClient()
            ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
            # 读取私钥
            ssh.connect(ip, port=22, username='root', key_filename='C:/Users/yy/.ssh/id_rsa')
            chan = ssh.invoke_shell()  # 新函数
            for com in self.command:
                if ip != "47.103.21.215" and "supervisorctl" in com and "celery_task:fill_proxy_pool" in com:
                    continue
                if ip == "47.103.21.215" and "supervisorctl" in com and "celery_task:fill_proxy_pool" not in com:
                    continue
                chan.send(com)
                if "supervisorctl" in com:
                    time.sleep(20)
                # 暂停一段时间，等待命令响应
                time.sleep(7)
                # print(f'执行的命令：{com}')
                res = chan.recv(20000) # 非必须，接受返回消息
                print(res.decode())
            ssh.close()
        except Exception as e:
            print(e)

    def run(self):
        for ip in self.linux_server:
            self.sshexecmd(ip)


if __name__ == '__main__':
    dep = Deployment()
    dep.run()
```


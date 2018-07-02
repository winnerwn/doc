ansible链接windows是借助winrm这个组件的，windows上需要开启winrm服务

winrm service 默认都是未启用的状态，先查看状态，如无返回信息，则是没启动

winrm enumerate winrm/config/listener

针对winrm service进行基础配置
winrm quickconfig

查看winrm service listener
winrm e winrm/config/listener

为winrm service配置auth:
winrm set winrm/config/service/auth '@{Basic="true"}'

为winrm service配置加密方式为允许非加密, 这个命令钥匙没有实现，暂时放一边
winrm set winrm/config/service '@{AllowUnencrypted="true"}'

防火墙要开启5985,5986这2个端口

[win]
10.10.10.3 ansible_user="Administrator" ansible_password="********" ansible_port=5985 ansible_connection="winrm" ansible_winrm_server_cert_validation=ignore

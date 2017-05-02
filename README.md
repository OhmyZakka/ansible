# learning-ansible

### Ansible简介
Ansible是个什么东西呢？官方的title是“Ansible is Simple IT Automation”——简单的自动化IT工具。
这个工具的目标有这么几项

- 自动化部署  
- 自动化管理配置项  
- 自动化的持续交互  
- 自动化云服务管理    

所有的这几个目标从本质上来说都是在一个台或者几台服务器上，执行一系列的命令而已。
通俗的说就是批量的在远程服务器上执行命令 。  
### Ansible工作机制
Ansible 在管理节点将 Ansible 模块通过 SSH 协议推送到被管理端执行，执行完之后自动删除，可以使用 SVN 等来管理自定义模块及编排.







### Ansible安装
除了源码编译安装还有两种比较简单的安装方式。   
#### 3.1 yum 安装
    yum -y intall ansible python-devel

#### 3.2 pip 安装
    pip ansible

### Ansible添加安全认证SSHKey

#### 4.1 利用Ansible实现无密钥登陆

因为密码在Inventory是明文配置，考虑安全性等其它原因，该方式只建议在首次添加SSHKey认证时使用，其它时间建议使用SSHKey认证方式。我们具体看下该方式如何实现。

**Step1: 配置Inventory，默认置/etc/ansible/hosts，添加配置如下:**

    定义db组   
	  [db]   
	  192.168.0.103    
	  192.168.0.110   
	  192.168.0.114    
  
   冒号分割，vars定义变量，改变db主机组的默认变量
   [db:vars]
  
    指定默认ssh用户为 admin   
    ansible_ssh_user="admin"
  
    指定ssh用户密码   
    ansible_ssh_pass="p@ssw0rd"    
  
**Step2: 测试默认变量是否生效，执行如下命令：**
	ansible db -m command -a 'whoami'   

**Step3: 调用Ansible authorized_key模块，添加认证至远程主机：**
	ansible db -m authorized_key -a "user=magedu key='{{ lookup('file', '/home/magedu/.ssh/id_rsa.pub') }}' path=/home/magedu/.ssh/authorized_keys manage_dir=no"   
  
或者调用Ansible或shell模块实现   
	#copy公钥至远程主机/tmp目录下
	ansible db -m copy -a "src=/home/magedu/.ssh/id_rsa.pub dest=/tmp/id_rsa.pub" 
	#添加公钥
	ansible db -m shell -a "cat /tmp/id_rsa.pub >> /home/magedu/.ssh/authorized_keys"  
  
**Step4: 验证是否可通过无密钥登陆，执行如下命令：**
	ssh admin@192.168.0.103   
#### 4.2 利用ssh-copy-id实现无密钥登陆 

命令ssh-copy-id用于复制指定用户的公钥至远程服务器并同时修改~/.ssh的目录权限。Linux入门必备，因其功能局限性，主机较多时频繁输入密码较麻烦，少量主机建议使用，具体方法如下：  

执行如下命令生成密钥对：

    /usr/bin/ssh-copy-id [-i [identity_file]] [user@]machine   
    
将本地的公钥copy至远程主机：

    ssh-copy-id -i /home/magedu/.ssh/id_rsa.pub admin@192.168.0.103

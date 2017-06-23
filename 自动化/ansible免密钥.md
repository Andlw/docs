一、ssh首次连接会出现会有keys连接提示，可以设置
    export ANSIBLE_HOST_KEY_CHECKING=False

二、生成管理主机的私钥和公钥
    ssh-keygen -t rsa -b 2048 -P '' -f /root/.ssh/id_rsa

三、添加主机信息到主机清单中
    #hosts
    [web]
    192.168.77.129 ansible_ssh_pass=1234.asd
    192.168.77.130 ansible_ssh_pass=asd.1234
    192.168.77.122 ansible_ssh_pass=aaa.aabb
    如果所有主机密码都一样的可以不指定密码，运行playbook时指定-k参数

    [all]
    10.153.5.68
    10.153.5.69
    10.153.5.70
    10.153.5.71

四、配置playbook
    #ssh-addkey.yml
    - hosts: all
      gather_facts: no

      tasks:

      - name: install ssh key
        authorized_key: user=root
                    key="{{ lookup('file', '/root/.ssh/id_rsa.pub') }}"
                    state=present

五、运行playbook
    ansible-playbook -u root -k -i hosts ssh_addkey.yml

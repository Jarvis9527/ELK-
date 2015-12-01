这里推荐一个修改使用了官方 elasticsearch.py 库的衍生版 GitHub地址见
https://github.com/Wprosdocimo/Elasticsearch-zabbix

ESzabbix.py
ESzabbix.userparm
ESzabbix_templates.xml
其中，前两个文件需要分发到每个 ES 节点上。如果节点上运行的是 yum 安装的 zabbix，二者的默认位置应该分别是：
/etc/zabbix/zabbix_externalscripts/ESzabbix.py
/etc/zabbix/agent_include/ESzabbix.userparm
然后在各节点安装运行 ESzabbix.py 所需的 python 库依赖：
yum install -y python-pbr python-pip python-urllib3 python-unittest2
yum -y install python-setuptools
pip install elasticsearch
安装成功后，你可以试运行下面这行命令，看看命令输出是否正常：
# /etc/zabbix/zabbix_externalscripts/ESzabbix.py cluster status

编辑agent /etc/zabbix/zabbix_agent.conf
UserParameter=ES[*], /server/script/ESzabbix.py cluster status $1

然后回到server上，
zabbix_get -s 192.168.10.74 -k ES   进程测试，看是否有返回值

0=>Green
1=>Yellow
2=>Red

#引用手册话方便理解
最后一个文件是 zabbix server 上的模板文件，不过在导入模板之前，还需要先创建一个数值映射，因为在模板中，设置了集群状态的触发报警，没有映射的话，报警短信只有 0, 1, 2 数字不是很易懂。
创建数值映射，在浏览器登录 zabbix-web，菜单栏的 Zabbix Administration 中选择 General 子菜单，然后在右侧下拉框中点击 Value Maping。
选择 create， 新建表单中填写：
name: ES Cluster State
0 ⇒ Green 1 ⇒ Yellow 2 ⇒ Red
完成以后，即可在 Templates 页中通过 import 功能完成导入 ESzabbix_templates.xml。
在给 ES 各节点应用新模板之前，需要给每个节点定义一个 {$NODENAME} 宏，具体值为该节点 elasticsearch.yml 中的 node.name 值。从统一配管的角度，建议大家都设置为 ip 地址。


其他

untergeek 最近刚更新了他的仓库，重构了一个 es_stats_zabbix 模块用于 Zabbix 监控，有兴趣的读者可以参考：https://github.com/untergeek/zabbix-grab-bag/blob/master/Elasticsearch/es_stats_zabbix.README.md

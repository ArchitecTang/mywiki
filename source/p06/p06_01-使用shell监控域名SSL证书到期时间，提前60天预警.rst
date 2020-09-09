6.1 使用shell监控域名SSL证书到期时间
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

最近遇到一个因为我们运营的域名太多，在管理过程中出现一个域名证书到期没有及时更换导致网站访问提示不安全的问题，所以需要写一个脚本对域名的SSL证书进行监控，配合zabbix进行预警(提前60天)，不多说直接上脚本内容：

6.1.1 脚本内容
''''''''''''''

::

    #!/bin/bash

    domain="$1"
    port="$2"
    echo  ${domain}:${port}| while read line;do # 读取传递的域名
        get_domain=$(echo "${line}" | awk -F ':' '{print $1}')
        get_port=$(echo "${line}" | awk -F ':' '{print $2}')

        # 使用openssl获取域名的证书情况，然后获取其中的到期时间
        END_TIME=$(echo | openssl s_client -servername ${get_domain}  -connect ${get_domain}:${get_port} 2>/dev/null | openssl x509 -noout -dates |grep 'After'| awk -F '=' '{print $2}'| awk -F ' +' '{print $1,$2,$4 }' )
        # 将日期转化为时间戳
        END_TIME1=$(date +%s -d "$END_TIME")
        # 将当前的日期转化为时间戳
        NOW_TIME=$(date +%s -d "$(date | awk -F ' +'  '{print $2,$3,$6}')")
        # 证书到期时间计算，到期时间减去目前时间再转化为到期天数
        RST=$(($(($END_TIME1-$NOW_TIME))/(60*60*24)))

        echo "${RST}"
    done

把脚本保存为文件存放在指定位置并赋予执行权限

::

    /ctdata1/scripts/ssl_domain_monitor/    #脚本存放路径自定义
    chmod +x get_https.sh

6.1.2 配置zabbix监控：
''''''''''''''''''''''

配置zabbix\_agent 配置文件，添加以下内容：

::

    UserParameter=domain_ssl_checks[*],/ctdata1/scripts/ssl_domain_monitor/get_https.sh $1

保存后重启zabbix\_agent

在zabbix\_server 控制面板上配置相应的 应用集、监控项、触发器即可

监控项的键值为：

::

    domain_ssl_checks[域名:端口]

监控数据更新时间 30s ##### 6.1.3 预警测试
调整域名的触发器，将<60天修改为<365天，等待30秒数据更新后会触发告警

我这里的测试结果为：

故障：

.. figure:: https://i.loli.net/2020/09/06/J9wbAcL3G6utUdI.png

恢复:

.. figure:: https://i.loli.net/2020/09/06/kySCHlsFje3V9cG.png

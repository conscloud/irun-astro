---
layout: '../../layouts/MarkdownPost.astro'
title: 'Openwrt状态信息推送到HA的两种实现方法'
pubDate: 2022-10-10
description: 'Openwrt软路由监控接入到HomeAssistant'
author: 'ZhJy'
cover:
    url: 'https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305272159687.png'
    square: 'https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305272159687.png'
    alt: 'cover'
tags: ["分享","OpenWRT","HomeAssistant"] 
theme: 'light'
featured: true

meta:
 - name: author
   content: ZhJy
 - name: keywords
   content: key3, key4

keywords: OpenWRT, HomeAssistant, HA, OP!
---

![](https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305272159687.png)

家里目前使用x86平台软路由作为主路由拨号，在群晖vmm虚拟机中安装了HAOS智能家居系统，HA中基于upnp的集成组件对路由器的监控项目不是很全，因此想将一些路由器的状态信息接入到HA中进行实时监控展示。这里介绍两种方法：

### 一、通过http模式推送

1、在进入HA系统-用户-长期访问令牌-创建令牌，新建一个长期访问令牌

2、写一个shell脚本openwrt_post.sh，获取路由器的状态信息，我这里取了CPU频率、温度、使用率、内存使用率、科学上网及开机时长。脚本如下：

```shell
#!/bin/bash

cpu_freq=$(cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_cur_freq|awk '{print ($1)/1000}')

temp_cpu=$(sensors|grep °C|sed -nr 's#^Core.*:.*\+(.*)°C .*#\1#gp'|sort -nr|head -n1'')

#cpu_freq=$(cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_cur_freq|awk '{print ($1)/1000}')

function getcpu(){

local AT=$(cat /proc/stat|grep "^cpu "|awk '{print $2+$3+$4+$5+$6+$7+$8 " " $2+$3+$4+$7+$8}')

sleep 1

local BT=$(cat /proc/stat|grep "^cpu "|awk '{print $2+$3+$4+$5+$6+$7+$8 " " $2+$3+$4+$7+$8}')

printf "%.01f" $(echo ${AT} ${BT}|awk '{print (($4-$2)/($3-$1))*100}')

}

cpu_used=$(getcpu)

mem_used=$(free -m|sed -n '2p'|awk '{print""($3/$2)*100}')

ssr_server=`cat /etc/config/shadowsocksr |grep global_server|awk -F\' '{print $2}'`

up_times=$(cat /proc/uptime| awk -F. '{run_days=$1 / 86400;run_hour=($1 % 86400)/3600;run_minute=($1 % 3600)/60;run_second=$1 % 60;printf("%d天%d时%d分%d秒\n",run_days,run_hour,run_minute,run_second)}')

post_data="{\"state\":\"$temp_cpu\", \"attributes\":{\"temp_cpu\":\"$temp_cpu\", \"cpu_freq\":\"$cpu_freq\", \"cpu_used\":\"$cpu_used\",\"mem_used\":\"$mem_used\",\"ssr_server\":\"$ssr_server\",\"up_times\":\"$up_times\"}}"

#echo $post_data

curl -X POST -H "Authorization: Bearer 第一步建的长期访问令牌" -H "Content-Type: application/json" -d "$post_data" http://192.168.XX.XX:8123/api/states/input_number.openwrtinfo
```

3、通过crontab任务调度或者写一个watch脚本循环调用，即可通过post形式将采集到的信息推送到HA。

```shell
#!/bin/bash
while :
do
        if ! ps | grep -w openwrt_post.sh | grep -v grep
        then                                    
                /opt/openwrt_post.sh
        sleep 60	#60秒取一次，可自行修改间隔
        fi                       
done
```

4、在HA中添加配置文件，将获取到的信息转为实体

```yaml
sensor:
  - platform: template
    sensors:
      openwrtinfo_temp_cpu:
        unit_of_measurement: °C
        value_template: "{{ states.input_number.openwrtinfo.attributes.temp_cpu }}"
      openwrtinfo_cpu_freq:
        unit_of_measurement: MHz
        value_template: "{{ states.input_number.openwrtinfo.attributes.cpu_freq }}"
      openwrtinfo_cpu_used:
        unit_of_measurement: "%"
        value_template: "{{ states.input_number.openwrtinfo.attributes.cpu_used }}"
      openwrtinfo_mem_used:
        unit_of_measurement: "%"
        value_template: "{{ states.input_number.openwrtinfo.attributes.mem_used }}"
      openwrtinfo_wan_sent:
        unit_of_measurement: GB
        unique_id: openwrtinfo_wan_sent
        value_template: "{{ states.sensor.openwrt_router_b_sent.state | multiply(1/1024/1024/1024) | round(2) }}"
      openwrtinfo_wan_received:
        unit_of_measurement: GB
        unique_id: openwrtinfo_wan_received
        value_template: "{{ states.sensor.openwrt_router_b_received.state | multiply(1/1024/1024/1024) | round(2) }}"
      openwrtinfo_ssr_server:
        value_template: "{{ states.input_number.openwrtinfo.attributes.ssr_server }}"
      openwrtinfo_up_times:
        value_template: "{{ states.input_number.openwrtinfo.attributes.up_times }}"
```

### 二、通过mqtt推送

1、先在openwrt中安装好mqtt，`mosquitto-client-nosll、libmosquitto-nossl`这两个包。

```shell
opkg update
opkg install mosquitto-client-nossl libmosquitto-nossl
```

2、内网已经部署了MQTT服务器。

3、监控脚本与方法一大部分相同，将最后一步的发送命令修改如下：

```shell
mosquitto_pub -r -L mqtt://mqtt:mqtt@192.168.XX.XX:1883/openwrtinfo -m "$post_data"
# mqtt:mqtt前面一个为mqtt用户名：mqtt密码
```

​		完整脚本如下：

```shell
#!/bin/bash

cpu1_freq=$(cat /sys/devices/system/cpu/cpufreq/policy0/scaling_cur_freq|awk '{print ($1)/1000}')
cpu2_freq=$(cat /sys/devices/system/cpu/cpufreq/policy1/scaling_cur_freq|awk '{print ($1)/1000}')
temp_cpu=$(sensors|grep °C|sed -nr 's#^Core.*:.*\+(.*)°C .*#\1#gp'|sort -nr|head -n1'')
#function getcpu(){
#	local AT=$(cat /proc/stat|grep "^cpu "|awk '{print $2+$3+$4+$5+$6+$7+$8 " " $2+$3+$4+$7+$8}')
#	sleep 1
#	local BT=$(cat /proc/stat|grep "^cpu "|awk '{print $2+$3+$4+$5+$6+$7+$8 " " $2+$3+$4+$7+$8}')
#	printf "%.01f" $(echo ${AT} ${BT}|awk '{print (($4-$2)/($3-$1))*100}')
#}
cpu_used=$(top -b -n 1|awk 'NR==2{print$8}'|awk '{print 100-$1}')
mem_used=$(free -m|sed -n '2p'|awk '{printf "%.2f",($3/$2)*100}')


#ip=$(curl -s https://api.ipify.org)
#for((y=0;y<${#array[*]};y++))
#do
#  if [[ "$ip" = "${array[y]}" ]];then
#  ssr_server=${array[y+1]}
#  fi 
#done
ssr_server=$(awk '/^'$(curl -s https://api.ipify.org)'/ {print $2}' /opt/myscripts/ssrserver)
hostname=$(cat /etc/config/system |grep hostname |awk -F\' '{print $2}')
cpu_brand=$(cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq)
cpu_arch=$(uname -m)
kernel=$(uname -r)
releases=$(echo "$(cat /etc/openwrt_release |sed -n '1p'|awk -F\' '{print $2}') $(cat /etc/openwrt_release |sed -n '6p'|awk -F\' '{print $2}') $(cat /etc/openwrt_version)")
boot_time=$(date -d "@$(( $(date +%s) - $(awk -F. '{print $1}' /proc/uptime) ))" +"%Y-%m-%d %H:%M:%S")

#ssr_server=`cat /etc/config/shadowsocksr |grep global_server|awk -F\' '{print $2}'`
up_times=$(cat /proc/uptime| awk -F. '{run_days=$1 / 86400;run_hour=($1 % 86400)/3600;run_minute=($1 % 3600)/60;run_second=$1 % 60;printf("%d天%d时%d分%d秒\n",run_days,run_hour,run_minute,run_second)}')

post_data="{\"temp_cpu\":\"$temp_cpu\", \"cpu1_freq\":\"$cpu1_freq\",\"cpu2_freq\":\"$cpu2_freq\", \"cpu_used\":\"$cpu_used\",\"mem_used\":\"$mem_used\",\"ssr_server\":\"$ssr_server\",\"up_times\":\"$up_times\",\"hostname\":\"$hostname\", \"cpu_brand\":\"$cpu_brand\", \"cpu_arch\":\"$cpu_arch\",\"kernel\":\"$kernel\", \"releases\":\"$releases\", \"boot_time\":\"$boot_time\"}"

#echo $post_data
#ssr_server=`cat /etc/config/shadowsocksr |grep global_server|awk -F\' '{print $2}'`
mosquitto_pub -r -L mqtt://mqtt:mqtt@mqtt.local:1883/openwrtinfo -m "$post_data"
```
4、在HA中增加配置文件如下：

```yaml
mqtt:
  sensor:
      - name: 'openwrt_cpu_tem'
        unique_id: openwrt_cpu_tem
        state_topic: 'openwrtinfo'
        unit_of_measurement: '°C'
        icon: 'mdi:thermometer'
        value_template: '{{ value_json.temp_cpu }}'        

      - name: 'openwrt_cpu1_freq'
        unique_id: openwrt_cpu1_freq
        state_topic: 'openwrtinfo'
        unit_of_measurement: 'MHz'
        icon: 'mdi:pulse'
        value_template: '{{ value_json.cpu1_freq }}'

      - name: 'openwrt_cpu2_freq'
        unique_id: openwrt_cpu2_freq
        state_topic: 'openwrtinfo'
        unit_of_measurement: 'MHz'
        icon: 'mdi:pulse'
        value_template: '{{ value_json.cpu2_freq }}'

      - name: 'openwrt_cpu_used'
        unique_id: openwrt_cpu_used
        state_topic: 'openwrtinfo'
        unit_of_measurement: '%'
        icon: 'mdi:cpu-64-bit'
        value_template: '{{ value_json.cpu_used }}'

      - name: 'openwrt_mem_used'
        unique_id: openwrt_mem_used
        state_topic: 'openwrtinfo'
        unit_of_measurement: '%'
        icon: 'mdi:memory'
        value_template: '{{ value_json.mem_used }}'
     - platform: mqtt
      - name: 'openwrt_ssr_server'
        unique_id: openwrt_ssr_server
        state_topic: 'openwrtinfo'

        icon: 'mdi:vpn'
        value_template: '{{ value_json.ssr_server }}'
     - platform: mqtt
      - name: 'openwrt_up_times'
        unique_id: openwrt_up_times
        state_topic: 'openwrtinfo'

        icon: 'mdi:clock-time-four-outline'
        value_template: '{{ value_json.up_times }}'

      - name: 'openwrt_name'
        unique_id: openwrt_name
        state_topic: 'openwrtinfo'
        value_template: '{{ value_json.hostname }}'

      - name: 'openwrt_cpu_brand'
        unique_id: openwrt_cpu_brand
        state_topic: 'openwrtinfo'
        value_template: '{{ value_json.cpu_brand }}'

      - name: 'openwrt_cpu_arch'
        unique_id: openwrt_cpu_arch
        state_topic: 'openwrtinfo'
        value_template: '{{ value_json.cpu_arch }}'
        
      - name: 'openwrt_kernel'
        unique_id: openwrt_kernel
        state_topic: 'openwrtinfo'
        value_template: '{{ value_json.kernel }}'

      - name: 'openwrt_releases'
        unique_id: openwrt_releases
        state_topic: 'openwrtinfo'
        value_template: '{{ value_json.releases }}'
        
      - name: 'openwrt_boottime'
        unique_id: openwrt_boottime
        state_topic: 'openwrtinfo'
        value_template: '{{ value_json.boot_time }}'

```

5、通过crontab任务调度或者写一个watch脚本循环调用，即可通过mqtt形式将采集到的信息推送到HA。

```shell
#!/bin/bash
while :
do
        if ! ps | grep -w openwrt_mqtt.sh | grep -v grep
        then                                    
                /opt/openwrt_mqtt.sh
        sleep 60	#60秒取一次，可自行修改间隔
        fi                       
done
```


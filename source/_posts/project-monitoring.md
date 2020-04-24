---
title: [monitoring]
date: 2020-01-21 14:30:19
tags: [shell,cron]
categories:
    - 프로젝트
thumbnail: ""
permalink: ""
---

## 1. 개요
 1. 여러개의 open source 가 있지만 간단한 내부 모니터 및 알람 툴이다.
 2. 클라이언트(view) : 웹, 아이폰, 안드로이드 앱(react-native 웹앱 구현), [view monitoring](http://116.126.87.167:10000)
 3. 서버(admin) : 
 4. 서버 환경 : centos6.9, mysql5.5, shell, php7.3
 5. 프로세스
<!-- more -->  
  5.1 어드민에서 프로젝트 등록 및 프로젝트 사용자 권한 설정을 한다.
  5.2 알람 어드민 툴에서 알람 임계치를 설정하여 (관심, 주의, 경고, 위험, 바로조치) sms를 보낼수 있다.
  5.3 모니터링 하고 싶은 서버에 설치 문서에 따라 설치 및 설정을 한다. 
  5.4 해당 시스템은 크론에 등록되어 매분 체크 된다.
  5.5 뷰에서 시스템 정보 및 알람 정보를 확인할수 있다.  
  5.6 기본적인 모니터링 셀 구현.
---

 ```bash
#!/bin/bash
root_path="/home/www/common_cron/alarm_admin";
if [ $# -gt 0 ]
then
	echo " Usage : ./monitor.sh"
	exit
fi
source $root_path/alarm_info.ini

active_ok=`$mysql_path -h$alarm_host -u$alarm_user -p$alarm_pwd $alarm_db -e "SELECT active_ok FROM svr_register WHERE pro_name = '$pro_name' AND svr_name = '$svr_name'"`
active_ok=`echo "$active_ok" | grep -E 'yes|no'`
if [ $active_ok = "no" ] || [ $active_ok = '' ]
then
	exit
fi

echo "pro_name : $pro_name, svr_name : $svr_name";
load_avg=`uptime | awk '{print $(NF-2)" "$(NF-1)" "$NF}'`
cpu=`/usr/bin/mpstat | tail -1 | awk '{print 100 - $12}'`
memory=`free | grep -i mem: | awk '{print $3/$2 * 100}'`
io_wait=`iostat -c|awk '/^ /{print $4}'`
disk=`df -h`
net_id=`netstat -i | awk '{print $1}' | grep -iv lo | grep -iv Iface | grep -iv Kernel`
for i in $net_id
do
	network_rx_error=`netstat -i | grep $i | awk '{print $5}'`
	network_tx_error=`netstat -i | grep $i | awk '{print $9}'`
done

START=$(date +%s)
if [ $svr_name == "db"* ] || [ $svr_name == "all" ]
then
#	echo "mysql -u$local_user -p'$local_pwd' $local_db < alarm_query.sql"
	mysql -u$local_user -p$local_pwd $local_db < alarm_query.sql
fi
END=$(date +%s)
query_sec=$(( $END - $START ))
http_cnt=`netstat -nap | grep :80 | grep ESTABLISHED | wc -l`
ping_sec=`ping -c 1 localhost | grep icmp_seq | awk '{print $8}'`
ping_sec=${ping_sec:5:10};

db_live_num=`ps -ef | grep mysql | egrep -v grep | wc -l`;
http_live_num=`ps -ef | grep httpd | egrep -v grep | wc -l`;

db_live_ok='yes';
if [ $db_live_num -lt 1 ]
then
	if [ $svr_name == "db"* ] || [ $svr_name == "all" ]
	then
		db_live_ok='no';	
	fi
fi

http_live_ok='yes';
if [ $http_live_num -lt 1 ]
then
	if [ $svr_name == "web"* ] || [ $svr_name == "all" ]
	then
		http_live_ok='no';
	fi
fi

$mysql_path -h$alarm_host -u$alarm_user -p$alarm_pwd $alarm_db -e "INSERT INTO system_log (pro_name, svr_name, load_avg, cpu, memory, io_wait, disk, network_rx_error, network_tx_error, query_sec, http_cnt, ping_sec, db_live_ok, http_live_ok, ip, joindate) VALUES 
('$pro_name', '$svr_name', '$load_avg', '$cpu', '$memory', '$io_wait', '$disk', '$network_rx_error', '$network_tx_error', '$query_sec', '$http_cnt', '$ping_sec', '$db_live_ok', '$http_live_ok', '127.0.0.1', now())"

 ```
# 2. alarm story board


# 3. alarm-admin, alarm-view 정보
# 4. 설치 및 설정
 /home/www/alarm_admin/_cron 모니터링 시스템에 설치 및 설정.
 alarm_info.ini 수정. - monitor.sh 레퍼
 - 크론 등록
####### alarm
##* * * * * /home/www/common_cron/alarm_admin/monitor.sh
##* * * * * /usr/bin/php /home/www/common_cron/alarm_admin/work_alarm.php
 
- sendPush.php - andriod
 ```php
         private function send_notification($tokens, $message, $re_num)
        {
                $url = $this->url;
                $fields = array(
                        'registration_ids' => $tokens,
                        'data' => $message
                );

                $key = $this->serverKey;
                $headers = array(
                        'Authorization:key =' . $key,
                        'Content-Type: application/json'
                );

                $ch = curl_init();
                curl_setopt($ch, CURLOPT_URL, $url);
                curl_setopt($ch, CURLOPT_POST, true);
                curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
                curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
                curl_setopt ($ch, CURLOPT_SSL_VERIFYHOST, 0);
                curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
                curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($fields));
                $result = curl_exec($ch);

                if ($result === FALSE) {
                        die('Curl failed: ' . curl_error($ch));
                }
                curl_close($ch);
                echo "Result : ".$re_num." ".$result."<br><br>\n\n";
                //return $result;
        }

 ```
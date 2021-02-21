nohup java -jar acm01.jar &
ps -ef | grep java
netstat -plntu|grep 9988
iptables -A INPUT -p tcp -m tcp --dport 9988 -j ACCEPT
iptables -nL --line-number

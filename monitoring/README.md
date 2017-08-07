 if the cadvisor metrics page http://node_ip:10255/metrics can't get host services cpu and mem metrics then try set
 any of system services CPU and Memory accounting like below:

systemctl set-property sshd.service CPUAccounting=yes
 systemctl set-property sshd.service MemoryAccounting=yes
 

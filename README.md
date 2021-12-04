## 1 Используя знания из лекции по systemd, создайте самостоятельно простой unit-файл для node_exporter

	root@ubuntu-20:~# cat /etc/systemd/system/node-exporter.service
	[Unit]
	Description=node-exporter service
	After=network.targer
	
	[Service]
	EnvironmentFile=-/etc/default/node_exporter
	ExecStart=/usr/local/bin/node_exporter $EXTRA_OPTS
	Restart=on-failure
	ExecStop=/usr/bin/killall node_exporter
	
	[Install]
	WantedBy=multi-user.target


## 2 Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.

	node_cpu_seconds_total{cpu="0",mode="idle"} 450.24
	node_cpu_seconds_total{cpu="0",mode="iowait"} 0.59
	node_cpu_seconds_total{cpu="0",mode="irq"} 0
	node_cpu_seconds_total{cpu="0",mode="nice"} 0
	node_cpu_seconds_total{cpu="0",mode="softirq"} 0.2
	node_cpu_seconds_total{cpu="0",mode="steal"} 0
	node_cpu_seconds_total{cpu="0",mode="system"} 2.56
	node_cpu_seconds_total{cpu="0",mode="user"} 1.65
	
	node_memory_Active_bytes
	node_memory_MemAvailable_bytes
	
	node_disk_io_now
	
	node_network_receive_bytes_total
	node_network_receive_drop_total
	node_network_receive_errs_total
	node_network_receive_packets_total
	node_network_transmit_bytes_total
	node_network_transmit_drop_total
	node_network_transmit_errs_total
	node_network_transmit_packets_total
	
## 4 Можно ли по выводу dmesg понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?

Да, можно. ОС осознаёт, что загружена на системе виртуализации. На Hyper-V:

	[    0.000000] Hypervisor detected: Microsoft Hyper-V
	
## 5 Как настроен sysctl fs.nr_open на системе по-умолчанию? Узнайте, что означает этот параметр. Какой другой существующий лимит не позволит достичь такого числа (ulimit --help)?

	root@ubuntu-20:~# sysctl fs.nr_open
	fs.nr_open = 1048576

Это максимальное количество файловых дескрипторов, которое может открыть процесс. Другой лимит:

	ulimit -n
	
Лимит можно изменить в файле:
	
	/etc/security/limits.conf 

Параметр: nofile

## 6 Запустите любой долгоживущий процесс (не ls, который отработает мгновенно, а, например, sleep 1h) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под PID 1 через nsenter.

	unshare -f --pid --mount-proc sleep 1h
	root@ubuntu-20:~# pgrep sleep
	3932
	root@ubuntu-20:~# nsenter --target 3932 --pid --mount
	root@ubuntu-20:/# ps aux
	USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
	root           1  0.0  0.0   8076   592 pts/0    S+   17:15   0:00 sleep 1h
	root          12  0.0  0.4   9836  4136 pts/1    S    17:19   0:00 -bash
	root          22  0.0  0.3  11476  3328 pts/1    R+   17:19   0:00 ps aux

## 7 Найдите информацию о том, что такое :(){ :|:& };:? Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?

Это forkbomb рекурсивно создаёт процессы в бесконечном цикле. В dmesg:

	cgroup: fork rejected by pids controller in /user.slice/user-1000.slice/session-5.scope

Для увеличения числа процессов нужно изменить значение pids.max в /sys/fs/cgroup/pids/user.slice/user-1000.slice/

	echo [num] > /sys/fs/cgroup/pids/user.slice/user-1000.slice/pids.max

	

# Facebook Workload

The facebook benchmark readme file can be found in [README.md](https://github.com/facebookarchive/oss-performance/blob/v2019.02.13.00/README.md)

### Preparing the VM
Facebook-Mediawiki workload needs specific network parameters to be set. Set the below params in `/etc/sysctl.conf` file.
```
net.ipv4.tcp_tw_reuse = 1
net.core.somaxconn = 1024
net.ipv4.ip_local_port_range = 1024 65000
```

Execute
```
systemctl daemon-reload
sysctl -p
```

Also if the CPU's speed is set to `ondemand` change it to `performance` by executing the below command
```
for file in /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor; do
	echo performance > $file
done
```

More info on CPU frequency can be found in [README.md](https://github.com/facebookarchive/oss-performance/blob/v2019.02.13.00/cpufreq.md)

### Starting the database server ####
To start the database server, you have to first `pull` the server image. To `pull` the server image use the following command:

    $ docker pull cloudsuite/web-serving:db_server

The following command will start the database server:

    $ docker run -dt --net=host --name=mysql_server cloudsuite/web-serving:db_server


### Starting the facebook workload benchmark ####
To start the facebook workload benchmark you have to first `pull` the server image. To `pull` the server image use the following command:

    $ docker pull cloudsuite/fb-workload:server
    
Create a file `cmd.sh` on the host which contains the command to run mediawiki workload. An example is present in `files/cmd.sh`
```
$ chmod +x cmd.sh
```

You can also use any one of the below parameters in `cmd.sh`
```
--hhvm=$HHVM_BIN
--php5=$PHP_CGI_BIN
--php=$PHP_FPM7_BIN
```

More info on parameters which can be provided to the facebook worload can be found in [README.md](https://github.com/facebookarchive/oss-performance/blob/v2019.02.13.00/README.md)

The following command will start the facebook workload:

```
$ docker run --cap-add sys_admin --net=host --name fb -v /<path>/cmd.sh:/oss-performance/cmd.sh cloudsuite/fb-workload:server
```

**Note:** Mysql max_connections should be set to >1000 for the mediawiki workload (add the below line in `cmd.sh`)
```
mysql -h10.52.2.161 -uroot -proot -e "SET GLOBAL max_connections = 1001;"
```
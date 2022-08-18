### Disk
```
lsblk
fdisk -l

cd /
mkdir minio-data
```


### Install Minio
```
sudo apt update
wget https://dl.min.io/server/minio/release/linux-amd64/minio

chmod +x minio

sudo mv minio /usr/local/bin

minio --version
```

### Configure Minio
```
sudo useradd -r minio-user -s /sbin/nologin

sudo chown minio-user:minio-user /usr/local/bin/minio

sudo chown minio-user:minio-user /minio-data -R

sudo mkdir /etc/minio

sudo chown minio-user:minio-user /etc/minio

vi /etc/default/minio


MINIO_ROOT_USER="minio"
MINIO_VOLUMES="/minio-data"
MINIO_OPTS="-C /etc/minio --address your_server_ip:9000 --console-address :9001"
MINIO_ROOT_PASSWORD="miniopassword"
```

### Configure Minio Service (systemd)
```
vi minio.service

[Unit]
Description=MinIO
Documentation=https://docs.min.io
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/usr/local/bin/minio
  
[Service]
WorkingDirectory=/usr/local/
  
User=minio-user
Group=minio-user
ProtectProc=invisible
  
EnvironmentFile=/etc/default/minio
ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_VOLUMES}\" ]; then echo \"Variable MINIO_VOLUMES not set in /etc/default/minio\"; exit 1; fi"
ExecStart=/usr/local/bin/minio server $MINIO_OPTS $MINIO_VOLUMES
  
# Let systemd restart this service always
Restart=always
  
# Specifies the maximum file descriptor number that can be opened by this process
LimitNOFILE=1048576
  
# Specifies the maximum number of threads this process can create
TasksMax=infinity
  
# Disable timeout logic and wait until process is stopped
TimeoutStopSec=infinity
SendSIGKILL=no
  
[Install]
WantedBy=multi-user.target

sudo mv minio.service /etc/systemd/system

sudo systemctl daemon-reload

sudo systemctl enable minio

sudo systemctl start minio

sudo systemctl status minio
```


```
1. Rocky Linux의 기본 AppStream에서 설치 및 실행

sudo dnf update -y
sudo dnf module enable mysql:8.0 -y
sudo dnf install @mysql -y

sudo systemctl start mysqld
sudo systemctl enable mysqld
sudo mysql_secure_installation
mysql -u root -p

```

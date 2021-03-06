# galeraCluster

MariaDB ve Galera Cluster

MariaDB, açık kaynaklı bir RDBMS yani ilişkisel veritabanı sunucusudur. MySQL’in geliştiricisi tarafından geliştirilmiştir. 
MySQL ile aynı altyapı üzerinde olduğundan dolayı MySQL ile uyumlu araçları ve komutları destekler. Bu nedenle MySQL kullanılan sistemlerden geçiş kolay olmaktadır.

# Kurulum ve Konfigürasyon İşlemleri

Kullanacağım sistem Ubuntu 20.04 LTS versiyonu, MariaDB’nin de 10.3 versiyonudur. Eğer MariaDB 10.1 ve 
üstü versiyonu ise Galera Cluster kurulum içerisinde gelmektedir.

# MariaDB kurulumu

Sırasıyla komutlar çalıştırılır.

- sudo apt-get install software-properties-common
- sudo apt-key adv –recv-keys –keyserver hkp://keyserver.ubuntu.com:80 0xF165F24C74CD1D8
- sudo add-apt-repository ‘deb [arch=amd64, arm64, ppc64el] ftp://ftp.ulak.net.tr/pub/MariaDB/repo/10.6/ubuntu bionic main’
- sudo apt update
- sudo apt install mariadb-server

MariaDB sunucusunu kurmuş bulunmaktayız. Sunucunun sağlıklı bir şekilde kurulduğunu teyit edelim.
- sudo systemctl status mariadb

Daha sonra sunucumuzda kurulu olan MariaDB’nin versiyonunu kontrol edelim.
- mysql -V

MariaDB’nin sağlıklı bir şekilde kurulduktan sonra güvenliği için kullanıcı ve sunucu ayarlarını yapalım.
- sudo mysql_secure_installation

Enter current password for root (enter for none): -> Daha şifremiz olmadığı için enter ile geçiyoruz

Set root password? [Y/n] -> Burada seçeneğimiz y. Root kullanıcısının şifresini belirleyelim. Daha sonra gelecek seçenekleri aşağıdaki gibi konfigüre ediyoruz.

Remove anonymous users? [Y/n] y

Disallow root login remotely? [Y/n] y

Remove test database and access to it? [Y/n] y

Reload privilege tables now? [Y/n] y

MariaDB kurulum ve konfigürasyon işlemleri tamamlandı. Veritabanı sunucumuza giriş yapıp işlemlerimizi 
gerçekleştirmeye başlayabiliriz. Arayüze erişim sağlayalım.
- mysql -u root -p 

Giriş yaptıktan sonra mevcut veritabanlarını listeleyelim

![image](https://user-images.githubusercontent.com/72556168/149827148-a5257066-b77c-45d6-9a51-20dae6c83063.png)

Bilgem adında yeni bir database oluşturalım.
create database bilgem;

Tekrar veritabanlarını listelersek oluşacak çıktı:

![image](https://user-images.githubusercontent.com/72556168/149827457-f8bbee36-cf18-4577-9877-53e646eafa6f.png)

# GALERA CLUSTER KURULUMU

Kurulumda kullanacağım node bilgileri

![image](https://user-images.githubusercontent.com/72556168/149827418-385085f1-7cc4-46b0-b127-51e6c689d48c.png)

Galera konfigürasyon dosyası oluştururak işimize başlayalım. 
- sudo nano /etc/mysql/mariadb.conf.d/galera.cnf

Dosya içerisinde cluster’ın adresini, özelliklerini, cluster’ı oluşturan node adreslerini ve isimlerini tanımlayacağız.
Yaptığım konfigürasyonlar aşağıdaki gibidir.

[mysqld]

bind-address=0.0.0.0

default_storage_engine=InnoDB

binlog_format=row 

innodb_autoinc_lock_mode=2


-Galera cluster konfigürasyonu

wsrep_on=ON

wsrep_provider=/usr/lib/galera/libgalera_smm.so wsrep_cluster_address="gcomm://10.8.130.223,10.8.129.16,10.8.128.254" 

wsrep_cluster_name="mariadb-galera-cluster" 

wsrep_sst_method=rsync 


-Cluster node konfigürasyonu

wsrep_node_address="10.8.128.254" 

wsrep_node_name="node1" 


# Node2 konfigürasyonu

[mysqld] 

bind-address=0.0.0.0

default_storage_engine=InnoDB

binlog_format=row

innodb_autoinc_lock_mode=2 


-Galera cluster configuration

wsrep_on=ON 

wsrep_provider=/usr/lib/galera/libgalera_smm.so

wsrep_cluster_address="gcomm://10.8.130.223,10.8.129.16,10.8.128.254"

wsrep_cluster_name="mariadb-galera-cluster"

wsrep_sst_method=rsync


-Cluster node configuration

wsrep_node_address="10.8.129.16"

wsrep_node_name="node2" 


# Node3 konfigürasyonu

[mysqld] 

bind-address=0.0.0.0

default_storage_engine=InnoDB

binlog_format=row

innodb_autoinc_lock_mode=2 


-Galera cluster configuration

wsrep_on=ON

wsrep_provider=/usr/lib/galera/libgalera_smm.so


wsrep_cluster_address="gcomm://10.8.130.223,10.8.129.16,10.8.128.254"

wsrep_cluster_name="mariadb-galera-cluster"

wsrep_sst_method=rsync


-Cluster node configuration

wsrep_node_address="10.8.128.254 "

wsrep_node_name="node3" 


Konfigürasyon dosyalarımızı hazırladıktan sonra ilk node’umuzda MariaDB servisini systemctl stop mariadb ile durduruyoruz. Ardından galera_new_cluster komutu ile beraber cluster’ımızı başlatıyoruz. Kalan iki node’umuzda mariadb servisini kapatıp açıyoruz. Böylelikle cluster’a katılmasını sağlıyoruz. Kontrol sağlayalım.

![image](https://user-images.githubusercontent.com/72556168/149827727-fd267278-5859-4477-875d-ec23e9fb75f7.png)













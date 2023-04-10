# **Ubuntu'da Sentinel Kullanarak Redis Kurulumu**
<html>
<head>
</head>
<body>

<p>Redis, bir ana kopya ile birden çok kopya arasında eşzamansız bir yöntem kullanarak veri çoğaltmayı destekler. Redis Cluster, karmaşık bir sistem için birden çok ana kopyayı  destekler. Burada, Ubuntu'da Sentinel kullanılarak High Availability' e sahip bir redisin nasıl kurulacağını gösteriyor.</p>


**İçindekiler**
<li><a href="Redis_kurulumu"><a href="#Redis_kurulumu">Redis Kurulumu</a></li>

<li><a href="Master_ayari"><a href="#Master_ayari">Master Ayarı</a></li>

<li><a href="Kurulum_cogaltma"><a href="#Kurulum_cogaltma">Kurulum Çoğaltma</a></li>

<li><a href="Master-Replica Kurulum Kontrol"><a href="#Master-Replica Kurulum Kontrol">Master-Replica Kurulum Kontrol</a></li>

<li><a href="Sentinel_Yapilandirma"><a href="#Sentinel_Yapilandirma">Sentinel Yapılandırma</a></li>

<li><a href="Sentinel_kurulum_kontrol"><a href="#Sentinel_kurulum_kontrol">Sentinel Kurulum Kontrol</a></li>

<li><a href="Kurulum Sırasında Karşılaşılabilinir Hatalar"><a href="#Kurulum_hatalari">Kurulum Sırasında Karşılaşılabilinir Hatalar</a></li>

<h2 id="Redis_kurulumu"></h2>

## **Redis Kurulumu**
<ol><li>Her sunucuya SSH üzerinden bağlanın.</li>
<li>Redis'i yüklemek için aşağıdaki komutları çalıştırın.</li>

```
$ curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

$ echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

$ sudo apt-get update

$ sudo apt-get install -y redis-server
```
<li>redis-server hizmetini etkinleştirin.</li>

```
$ systemctl enable redis-server
```
</ol>

<h2 id="Master_ayari"></h2>

## **Master Ayarı**

<ol><li>Terminalde <span style="background-color:green; color:white; padding:3px;border-top-right-radius: 10px;
border-bottom-left-radius: 10px;">/etc/redis/redis.conf </span> dosyası açılır.</li>
<li>Aşağıdaki yapılandırmayı bulun, yorumları kaldırın ve şekildeki gibi düzenleyin. İsteğe bağlı requirepass ve masterauth da ekleme yapılabilir. Yönetici izni girişi ve şifre sağlar.</li><br>

```
bind 0.0.0.0

requirepass "<YOUR_PASSWORD>"

masterauth "<YOUR_PASSWORD>"
```
<li>Redis sunucu hizmetini yeniden başlatın.

```
$ sudo service redis-server restart
```
</li></ol>

<h2 id="Kurulum_cogaltma"></h2>

## **Kurulum Çoğaltma**
<ol><li>Birden çok port üzerinde redis çalıştırılmak istenir ise, ayrı ayrı dosya açılabilir, yükleme ile beraber default olarak gelen config dosyasını oluşturulan dosyalara kopyalayıp, ayarlamaları istenilen portlara göre değiştirilebilinir.</li>
<li>Kopyası dağıtılmak istenilen config dosyası açılır. <span style="background-color:green; color:white; padding:px;border-top-right-radius: 10px;
border-bottom-left-radius: 10px; padding:2px;">/etc/redis/redis.conf</span><br></br>

```
bind 0.0.0.0

requirepass "<YOUR_PASSWORD>"

masterauth "<YOUR_PASSWORD>"

replicaof <MASTER_NODE_IP> 6379
```
</li>
<li>Redis-sunucu hizmetlerini yeniden başlatın.

```
$ sudo service redis-server restart
```
</li></ol>

<h2 id="Master-Replica Kurulum Kontrol"></h2>

## **Master-Replica Kurulum Kontrol**
<p>Bu bölümde, ana kopyanın kurulumunun beklendiği gibi çalışıp çalışmadığının nasıl kontrol edileceği gösterilmektedir.</p>
<ol><li>Redis-server'ın günlüğü kontrol edilir.

```
$ tail -f /var/log/redis/redis-server.log
```
</li>
<li>Redis sunucusuna bağlanmak için Redis CLI çalıştırılır.</li>

```
$ redis-cli
```
<li>Kimlik doğrulaması için aşağıdaki komut çalıştırılır.</li> 

```
$ AUTH <YOUR_PASSWORD>
```

<li>Redis çoğaltma bilgilerini kontrol etmek için aşağıdaki komut çalıştırılır.</li>

```
$ INFO REPLICATION
```

<li>Örnek bir sonuç: <span style="background-color:green; color:white; padding:3px;border-top-right-radius: 10px;
border-bottom-left-radius: 10px;"> connected_slaves:2</span> her iki replica'nın master'a başarı ile bağlandığını gösterir.

```
# Replication

role:master

connected_slaves:2

slave0:ip=139.180.217.55,port=6379,state=online,offset=14,lag=1

slave1:ip=139.180.156.164,port=6379,state=online,offset=14,lag=1

master_failover_state:no-failover

master_replid:43855ff334c072ee5a3f4db39b1b27b31f55a3a4

master_replid2:0000000000000000000000000000000000000000

master_repl_offset:14

second_repl_offset:-1

repl_backlog_active:1

repl_backlog_size:1048576

repl_backlog_first_byte_offset:1

repl_backlog_histlen:14
```
</li>

<li>Master'da bir key ayarlamak için <span style="background-color:green; color:white; padding:2px;border-top-right-radius: 10px;
border-bottom-left-radius: 10px;"> SET </span>komutu çalıştırılır.</li>

```
$ SET hello world
```

<li>Redis çoğaltma sunucunuza bağlanın.</li>
<li>Redis sunucusuna bağlanmak için Redis CLI'i çalıştırılır.</li>

```
$ redis-cli -p <PORT>
```
<li>Kimlik doğrulaması için aşağıdaki komut çalıştırılır.

```
$ AUTH <YOUR_PASSWORD>
```

<li>Key değerini almak için aşağıdaki komut çalıştırılır. Bu işlem ile master'dan alınan verinin başarılı bir şekilde kopyalandığı görülür. </li>

```
$ GET hello
```
</li></ol>

<h2 id="Sentinel_Yapilandirma"></h2>

## **Sentinel Yapılandırma**
Bu bölüm, önceki bölümde ana kopya kurulumunu izlemek için Redis Sentinel'in nasıl kurulacağını gösterir.
Redis Sentinel, farklı sunucularda çalışan birden çok Sentinel işleminin bulunduğu dağıtılmış bir sistemdir. Bu, sistemin arızalara karşı dayanıklı olmasını sağlar. Herhangi bir kesinti senaryosuna karşı, sentinel yeni master olması için replica'yı izler.

<ol><li>Her bir sunucu için Redis Sentinel'i kurun.

 ```
 $ sudo apt install redis-sentinel
 ```
</li>

 <li>Başlatmak için Redis-Sentinel hizmetini etkinleştirin.

```
$ systemctl enable redis-sentinel
```
</li>

<li>Yapılandırma dosyasını açın. <span style="background-color:green; color:white; padding:5px;border-top-right-radius: 10px;
border-bottom-left-radius: 10px;">/etc/redis/sentinel.conf </span></li><br>

<li>Satırları bulun ve aşağıdaki gibi düzenleyin.

```
sentinel monitor mymaster <MASTER_NODE_IP> 6379 2

sentinel auth-pass mymaster <YOUR_PASSWORD>

sentinel down-after-milliseconds mymaster 5000

sentinel failover-timeout mymaster 60000

protected-mode no
```
</li>


<li>Redis Sentinel hizmetini yeniden başlatın.

```
$ sudo service redis-sentinel restart
```
</li>
</ol>


<h2 id="Sentinel_kurulum_kontrol"></h2>

## **Sentinel Kurulum Kontrol**
Bu bölüm de, Redis Sentinel'in beklendiği gibi çalışıp çalışmadığının nasıl kontrol edileceği gösterilmektedir.

<ol><li>Redis Sentinel günlüğünü kontrol edin.

```
$ tail -f /var/log/redis/redis-sentinel.log
```
</li>
<li>Redis Sentinel sunucusuna bağlanmak için Redis CLI'yi çalıştırın.

```
$ redis-cli -p 26379
```
</li>
<li>Redis çoğaltma bilgilerini kontrol etmek için aşağıdaki komut çalıştırılır.

```
$ INFO SENTINEL
```

</li>

<li>Örnek sonuç:

```
# Sentinel

sentinel_masters:1

sentinel_tilt:0

sentinel_tilt_since_seconds:-1

sentinel_running_scripts:0

sentinel_scripts_queue_length:0

sentinel_simulate_failure_flags:0

master0:name=mymaster,status=ok,address=45.32.121.40:6379,slaves=2,sentinels=3
```
</li>

<li>Sentinelin hangi porta bağlı olduğunu öğrenmek için aşağıdaki komutu çalıştırın.

```
SENTINEL get-master-addr-by-name mymaster
```
</li></ol>

<h2 id="Kurulum_hatalari"></h2>

## **Kurulum Sırasında Karşılaşılabilinir Hatalar**

<ul type="circle"><li><u><i>BIND ADDRESS ALREADY IN USE</u></i></br></ul>

``` 
ps aux | grep redis
``` 
komutunu çalıştırıp, hangi portların, sentinellerin çalıştığı görüntülenir. Sorun olan sunucuyu <span style="background-color:green; color:white; padding:2px;border-top-right-radius: 10px;
border-bottom-left-radius: 10px;"> kill -9 PID</span> komutu ile parent ıd öldürülür. Böylece sunucu tekrar çalıştırılabilinir.

<ul type="circle"><li><u><i>CANNOT RESTART REDIS-SENTINEL UNIT</u></i></br></ul>

```
journalctl -u redis-sentinel
```

komutunu çalıştırıp, hata mesajlarını takip ederek sorun çözülebilinir.

```
sudo service --status-all
```
komutu ile aktif çalışan ve çalışmayan servisler listelenir.


<ul type="circle"><li><u><i>SUDO DOESN'T WORK</u></i></br></ul>
Terminale ulaşılamama sorununda aşağıdaki komut çalıştırılarak gerekli izinler verilebilinir.

```
su - root

chmod 440 /etc/sudoers
```

<ul type="circle"><li><u><i>REDIS-SENTINEL -DUPLICATE ERROR</u></i></br></ul>
İlgili sentinel config dosyasında "sentinel monitor mastername ıp port quorum"  satırından birden fazla olabilir.

<ul type="circle"><li><u><i>HOW TO CONFIGURE REDIS SENTINEL LOG FILE LOCATION</u></i></br></ul>
İlgili sentinel'in config dosyasında logfile "/var/log/redis/redis-sentinel.log" olarak yazılmalıdır.

<ul type="circle"><li><u><i>REDIS-SENTINEL VE REDIS-SERVER SERVİSLERİNİN ÇALIŞIP ÇALIŞMADIĞI AŞAĞIDAKİ KOMUT ÇALIŞTIRILARAK KONTROL EDİLEBİLİR</u></i></br></ul>

```
systemctl status redis-server
systmectl status redis-sentinel
```

<ul type="circle"><li><u><i>COULD NOT CONNECT TO REDIS AT 127.0.0.0.0:6379 : CONNECTION REFUSED</u></i></br></ul>


<ul type="circle"><li><u><i>EĞER CONFIG DOSYALARINDA CLUSTER-ENABLED YES İSE YORUM SATIRINA ALINMASI GEREKİR.</u></i></br></ul>

<ul type="circle"><li><u><i>HERHANGİ BİR SERVİSİN FAİL OLMASI YA DA KONUMUNUN BULUNAMAMASI DURUMUNDA AŞAĞIDAKİ KOMUT ÇALIŞTIRILARAK SİLİNİP, TEKRAR YÜKLENİR.</u></i></ul>


```
systemctl --type=service | grep sentinel
ps -ef
ps -ef | grep sentinel
```
komutları ile PID'ler öğrenilir. Sentinel servisi görüntülenir. Aşağıdaki komut ile silinir.
```
sudo apt-get remove redis-sentinel.service
```
<ul type="circle"><li><u><i>EN SON MASTER'IN KOPYASINI REPLICA'LARA DOĞRU BİR ŞEKİLDE DAĞITIP, SENTINEL'LERIN DÜZGÜN BİR ŞEKİLDE BUNU SAĞLAMASINI ŞU ŞEKİLDE TEST EDEBİLİRİZ:</u></i></ul>

```
kill -9 MASTERPID
```
master olan sunucunun redis PID'sini öldürüp, diğer kalanlardan birinin sentinel tarafından master seçilip, diğerlerinin slave olmasının kontrolünün sağlanması gerekir. Aynı zamanda yeni seçilen master'a veri girişi yapılabiliyor olunması gerekiyor.

</body>
</html>

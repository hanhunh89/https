# https를 만들어보자

# 들어가면서

http는 태생적으로 보안을 고려하지 않고 만들었다.<br>
computer science에서 사용하는 거의 모든 프로토콜이 그렇다.<br>
이 험난한 세상, https를 써서 통신을 암호화해보자. 

# 환경
google ec2 "Debian GNU/Linux 11 (bullseye)"
apache

# https 작동 원리
https를 쓰는 이유는 두가지다. <br>

첫번째는, 접속하는 사이트가 안전한 사이트인지 인증<br>

두번째는, 통신내용의 암호화다..<br>

https를 이용하여 서버에 접속할 때, 인증서를 받는다.<br> 
우리는 이 인증서를 통하여 해당 사이트가 공식 홈페이지를 사칭하는 사이트가 아닌 실제 사이트인것을 확인할 수 있다.<br>
또한 서버와 대칭키를 공유하여 통신내용을 암호활 수 있다. 

자세한 https 작동 원리는 https://nuritech.tistory.com/m/25 여기에 매우 자세하게 설명되어있다.

# https 구축

## apache 설치
web-was-db 구조에서 https는 주로 web에서 처리한다.<br>
우리는 apache를 사용해보자. <br>
```
sudo apt-get update
sudo apt-get install apache2
```
아파치를 설치했다.<br><br>
```
$ netstat -natlp
mamdjango3@instance-1:/etc$ sudo netstat -natlp
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp6       0      0 :::80                   :::*                    LISTEN      81655/apache2       
```
apache가 80으로 listen하고 있다. 

# 개인키 생성
```
openssl genpkey -algorithm RSA -out private_key.pem
```
private_key.pem라는 개인키가 생성되었다. 이것으로 인증서를 만들자.<br>

# CSR (Certificate Signing Request) 생성
```
openssl req -new -key private_key.pem -out csr.pem
```

# 인증서 생성
```
openssl x509 -req -in csr.pem -signkey private_key.pem -out certificate.pem
```
실제 서비스를 할 때는 공인 인증기관에서 인증서를 발급받아야 한다. <br>
하지만 내가 인증기관이 되서 인증서를 만들것이기 때문에... 대충 만들어주자. <br>

# 아파치 mod_ssl 설정
아파치에는 다양한 모드가 있다. 추가 기능을 위한 모듈이라 생각하면 된다. <br>
https를 위해서는 ssl이 필요하다. 확인해보자.
```
$ls -al /etc/apache2/mods-enabled
```
위 명령어로 enable된 mod를 확인할 수 있다. ssl은 없다.<br>
enable시켜주자.
```
$ sudo a2enmod ssl
$ls -al /etc/apache2/mods-enabled
```

이제 /etc/apache2/site-available에서 conf 파일을 만들어주자. <br>
아파치 서비스를 위한 설정파일이라 생각하면 된다.<br>
/etc/apache2/sites-available 디렉토리에 가면 이미 default-ssl.conf 파일이 있다. <br>
이 파일을 적당히 수정해서 my-ssl.conf 파일을 만들어주자
```
#/etc/apache2/sites-available/my-ssl.conf
<IfModule mod_ssl.c>
        <VirtualHost _default_:443>
                ServerAdmin webmaster@localhost

                DocumentRoot /var/www/html/my-ssl


                ErrorLog ${APACHE_LOG_DIR}/error.log
                CustomLog ${APACHE_LOG_DIR}/access.log combined

                SSLEngine on

                SSLCertificateFile      /home/mamdjango3/certificate.pem
                SSLCertificateKeyFile  /home/mamdjango3/private_key.pem

                <FilesMatch "\.(cgi|shtml|phtml|php)$">
                                SSLOptions +StdEnvVars
                </FilesMatch>
                <Directory /usr/lib/cgi-bin>
                                SSLOptions +StdEnvVars
                </Directory>
        </VirtualHost>
</IfModule>
```
이제 /var/www/html/my-ssl/index.html파일을 만들어주자.<br>
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello Page</title>
</head>
<body>
    <h1>Hello!</h1>
</body>
</html>
```
default.conf 적용 해제
```
sudo a2dissite 000-default.conf 
```
my-ssl.conf 파일 적용 
```
sudo a2ensite my-ssl.conf
```
apache 재시작
```
sudo service apache2 restart
```

# https 테스트
```
https://{server_ip}
```
![Example Image](https://raw.githubusercontent.com/사용자명/리포지토리명/브랜치/images/example.png)

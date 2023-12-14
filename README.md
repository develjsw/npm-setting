# NPM(Nginx, PHP, MariaDB) Setting 방법

## VirtualBox와 Vagrant정의
1. VirtualBox란 무엇인가? 
   - 하드웨어를 소프트웨어적으로 구현해서 그 위에서 운영체제가 작동하도록 도와주는 소프트웨어입니다.
2. Vagrant란 무엇인가?
   - 가상머신을 쉽게 만들고 관리해주는 툴로 가상환경 위에 가상머신을 설치하고 관리해주는 툴입니다.

## VirtualBox와 Vagrant를 통한 CentOs7에서의 NPM설치 및 설정

### VirtualBox와 Vagrant설치 및 설정
[ 1 ] VirtualBox(가상 머신)설치<br>
    - https://www.virtualbox.org/wiki/Downloads (필자는 VirtualBox 6.1.18 platform packages -> Windows hosts로 설치 하였다.)

[ 2 ] 가상머신 확장팩 설치 및 추가<br>
    - https://www.virtualbox.org/wiki/Downloads 에서 All supported platforms를 설치 해준다. (확장팩 설치)<br>
    - VirtualBox실행 -> 도구 -> 환경설정 -> 확장 -> 우리가 다운 받았던 Oracle_VM_VirtualBox_Extension_Pack-6.1.18파일을 올려준다.

[ 3 ] Vagrant설치<br>
    - https://www.vagrantup.com/downloads 에서 vagrant설치 (필자는 windows 64bit로 설치 하였다.)
    - vagrant를 설치하면 디폴트 폴더는 C:\HashiCorp\Vagrant이고 그 이후에 vagrant를 어디서 실행하든 상관은 없다. <br>그러나 가급적이면 workspace디렉토리를 만들어서 그 아래에 project폴더를 만든 후 vagrant를 실행하길 권장한다.<br>
    ex) D:\workspace\project에서 vagrant 실행

[ 4 ] Vagrantfile생성 및 Vagrant설정 및 실행<br>
    - vagrant를 이용하여 virtualbox에 vagrant box추가를 해야하는데 이미 누군가가 설정해두고 검증된 box를 가져와서 사용하면 편하다. <br>주소는 https://app.vagrantup.com/boxes/search 이고
    (필자는 centos7를 사용할 것이기에 centos/7로 검색하였고 최상단에 가장 많이 다운 받은 것을 사용 할 예정이다.)
    사용할 box를 클릭 해보면 Vagrantfile설정 값이 나와 있는데 이 부분을 참조하여 Vagrantfile파일에 넣어주면 된다.
    - CLI에서 본인이 작업할 프로젝트로 이동 후 vagrant init명령어를 실행하면 Vagrantfile이 생성 되었을 것이다.<br>
    이 파일을 편집기/IDE로 열고 위에서 언급했던 값을 참조하여 아래와 같이 작성 후<br><br>

   
   ~~~  
   Vagrant.configure("2") do |config|
   config.vm.box = "centos/7"
   config.vm.network "private_network", ip: "192.168.33.10"					                                    
   config.vm.synced_folder ".", "/vagrant", type: "virtualbox", mount_options: ["dmode=777", "fmode=777"]
   end
   
   # 설명 
   # config.vm.box => 박스 설정값 (centos7)
   # config.vm.network => 설정하고 싶은 아이피 설정
   # config.vm.synced_folder => host os와 gest os의 싱크를 맞춰주는 부분으로 host os에서 해당 위치의 폴더에 이미지를 올리면 gest os에도 바로 반영이 되는 것이 확인 가능함
   ~~~ 
  vagrant up 명령어를 실행해준다.<br> 이때 vagrant up을 최초 실행 했을 경우 우리가 설정했던 값으로 os설치 및 네트워크 설정 host os gest os싱크 설정 등등이 진행되므로 시간이 조금 걸린다.

[ 5 ] Vagrant ssh접속 또는 putty에서 ssh접속
   - 이제 우리는 vagrant ssh를 통해 접속이 가능하며 ip addr명령어를 통해 우리가 설정한 ip를 확인 할 수 있으며 ping 8.8.8.8(구글 퍼블릭 아이피)/ ping 우리가 설정한 아이피를 날려보는 행위를 통해서 외부와의 통신도 가능하다는 것을 확인 할 수 있다.
   - putty를 키고 우리가 설정한 아이피를 입력하고 예를 누르고 root를 입력했을때 'No supported authentication methods available (server sent: publickey.gssapi-keyex,gssapi-with-mic)'에러가 뜨는 것으르 확인 할 수 있는데<br> 이는 sshd설정에서 
   ssh에 비밀번호로 접속하는 부분이 막혀있기 때문이므로<br> 
   cmd창에서 vagrant ssh에 접속하여 su/vagrant로 root계정으로 진입 후<br>
   vi /etc/ssh/sshd_config에서 PasswordAuthentication yes주석을 해제한 후 저장 -> systemctl restart sshd.service 또는 service sshd restart명령어를 통해 재시작을 해준 후 다시 putty를 통해 접속을 해보면 접속이 가능한 것을 확인 할 수 있다.    

[ 6 ] 추가 참고 사항
   - Vagrantfile에 설정한 아이피가 VirtualBox의 Ethernet Adapter에 설정되어 있는 아이피 범위가 아닐 경우 VirtualBox Interface알림 창이 2번 뜨는데 동의 하면 문제 없이 작동한다. 
   - vagrant up명령어 실행시 VirtualBox를 보면 프로젝트폴더명_default_숫자의 세션이 하나 생성되어 ex) project_default_1642131231121_41214 우리가 설정했던 설정값으로 os설치 및 설정값 셋팅을 하는 것을 확인 할 수 있다. 
   - 해당 세션 마우스 우클릭 -> 설정 -> 저장소에는 vmdk가 올라와져 있는 것을 확인할 수 있다. 
   - 해당 세션 마우스 우클릭 -> 설정 -> 네트워크에 어댑터1 NAT, 어댑터2에는 vagrantfile에서 설정했던 config.vm.network에 아이피 범위에 해당하는 Ethernet Adapter가 있을 경우에는 그 어댑터와 연결되고 범위에 없을 경우에는 새로 생성하여 연결되어 있다. ex) VirtualBox Host-Only Ethernet Adapter #3
   - 공유폴더에는 우리가 Vagrantfile에 config.vm.synced_folder에서 설정한 폴더의 위치가 설정되어 있음을 확인 할 수 있다.
   - host os와 gest os의 sync가 정상 작동하는지 확인 하려면 host os에서 우리가 생성한 프로젝트 폴더로 이동하여 이미지를 생성하고 gest os에 putty나 vagrant ssh를 통해 접속하여 cd /vagrant로 이동하여 확인이 가능하다.
   - host os를 끄기전에 vagrant halt를 통해 vagrant를 종료 시켜주고 끄는 것이 좋다.


### NPM 설치 전 yum update
~~~
$ yum update
~~~

### Nginx 설치 및 기본 설정 (php-fpm연동 시 추가 수정사항 존재)
[ 1 ] 폴더이동 
~~~
$ cd /etc/yum.repos.d/ 
~~~
[ 2 ] nginx 레포파일 생성
~~~
$ vi nginx.repo
~~~
[ 3 ] 아래 내용 입력 후 저장
~~~
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
~~~
[ 4 ] nginx 설치
~~~
$ yum install -y nginx
~~~
[ 5 ] 설치된 nginx version 확인
~~~
$ nginx -v
~~~
[ 6 ] nginx 시작
~~~
$ systemctl start nginx
~~~
[ 7 ] nginx 상태확인 (active running)으로 뜨면 정상적으로 실행중인 상태 
~~~
$ systemctl status nginx
~~~
[ 8 ] 위에 우리가 설정한 아이피로 접속<br> 
~~~
ex) 192.168.33.10
~~~
[ 9 ] nginx server root폴더 변경 (본인이 사용할 디렉토리로 정하면 됨. 필자는 /vagrant/www/public 위치로 잡을 예정)
~~~
$ vi /etc/nginx/conf.d/default.conf
~~~

아래와 같이 수정 후 저장
~~~
location {
     root /vagrant/www/public;
}
~~~
~~~
$ cd /vagrant
~~~
~~~
$ mkdir www
~~~
~~~
$ cd www
~~~
~~~
$ mkdir public
~~~
~~~
$ cd public
~~~

임시파일 생성 (아래와 같이 명령어 입력 후 내용 저장)  
~~~
$ vi index.html
~~~

[ 10 ] 여기까지 진행 후 해당 아이피로 접속을 하면 403 forbidden에러가 뜰텐데 이 부분은 setenforce 0 명령어를 통해 해결이 가능하다.<br>
    centos7의 레드햇계열 리눅스부분부터 보안이 강화되어 SELinux부분을 임시 비활성화 해준 것
~~~
$ setenforce 0
~~~

[ 11 ] nginx 자동 재시작 설정
~~~
$ systemctl enable nginx
~~~

[ 12 ] nginx 설정파일 권한 변경
~~~
$ chown root. /etc/nginx/nginx.conf
~~~

### yum 패키지 다운로드 및 저장소 추가
[ 1 ] yum 패키지인 wget 다운로드<br>
   (리눅스 기본명령어 - wget # 'Web Get'의 약어로 웹 상의 파일을 다운로드 받을 때 사용하는 명령어로 wget은 비 상호작용 네트워크 다운로더 이다.
   즉, 네트워크 상에서 데이터를 다운로드하는 기능을 수행한다.)
~~~
$ yum -y install wget
~~~

[ 2 ] 레드햇7(centos7)에 epel-release-latest 7 추가
~~~
$ wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
~~~

EPEL(Extra Packages for Enterprise Linux)은 Fedora Project에서 제공되는 저장소로 각종 패키지의 최신 버전을 제공하는 community 기반의 저장소다.<br>
RHEL의 패키지 정책은 보수적이고 안정성 위주라 패키지 업데이트가 잘 되지 않는다.<br>
최신 버전의 패키지를 사용하고 싶은 경우에는 EPEL이나 REMI 등의 repository를 등록하고 설치/update를 하시면 된다.<br>
 
[ 3 ] ius 외부 저장소 추가
~~~
$ wget https://repo.ius.io/ius-release-el7.rpm
~~~

IUS는 클라우드 호스팅 서비스인 Rackspace의 지원을 받는 2명의 전문 엔지니어가 관리하고 있다.<br>
외부 저장소 중에서는 가장 전문적이고 보수적이라 믿을만하다.

[ 4 ] RPM(Redhat Package Manager) -ivh(설치/자세한 정보 출력/설치 진행상황을 #문자를 이용하여 출력)명령어로 위에 설치 상황 보기
~~~
$ rpm -ivh epel-release-latest-7.noarch.rpm
$ rpm -ivh ius-release-el7.rpm
~~~

### PHP 설치 및 설정
[ 1 ] 버전은 본인이 원하는 버전으로 설치하면 된다. (필자는 7.3버전으로 진행하였음.)
~~~
$ yum --enablerepo=ius-archive install -y php73*
~~~

( yum --enablerepo=[저장소] install [패키지] 여러개의 yum repository가 있을 경우 사용할 repos 를 지정한다. )

[ 2 ] php-fpm설정파일 권한 변경
~~~
$ chown root. /etc/php-fpm.d/www.conf
~~~

[ 3 ] php-fpm 시작/자동 재시작
~~~
$ systemctl start php-fpm
$ systemctl enable php-fpm
~~~

### mariaDB 설치 및 설정
[ 1 ] mariaDB 레포 파일들이 존재 하는 디렉토리로 이동
~~~
$ cd /etc/yum.repos.d
~~~

[ 2 ] mariaDB 레포파일 생성
~~~
$ vi MariaDB.repo
~~~

[ 3 ] 아래 내용 입력 후 저장  
~~~
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.4/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
~~~

[ 4 ]. yum목록 캐시 업데이트
~~~
$ yum makecache fast
~~~

[ 5 ] mariaDB 설치 
~~~
$ yum -y install MariaDB-server MariaDB-client
~~~

[ 6 ] mariaDB 시작/자동 재시작
~~~
$ systemctl start mariadb
$ systemctl enable mariadb
~~~

여기까지 진행한 후 mysql -uroot -p를 통해 접속해보면 (password는 입력하지 않고 enter하거나 vagrant를 입력해주면 됨) 접속이 가능한 것을 확인 할 수 있으며<br>
test테이블이 디폴트로 생성되어 있음을 확인 할 수 있다.<br>

[ 7 ] DB Tool로 접속
heidSql과 같은 DB Tool로 접속하고자 할때 Access denied for user 'root'@192.168.33.1 (using password: YES)와 같은 접근 불가 오류가 발생한다.<br>
이때는 mariaDB에 접속하여 권한을 열어 줄 수 있는 명령어인 grant all privileges on 디비명.* to 아이디@아이피 identified by '패스워드'의 명령어를 사용해줘야 한다.<br>
mariaDB에 접속하지 않고 server에서 아래와 같이 명령어를 입력하면 접근이 가능해진다.
~~~
mysql -e "create database 디비명"
mysql -e "create user '아이디'@'localhost' identified by '패스워드'"						
mysql -e "GRANT USAGE ON *.* TO '아이디'@'%' IDENTIFIED BY '패스워드'"					
mysql -e "grant all privileges on 디비명.* to '아이디'@'%' identified by '패스워드'" # 해당 디비에대한 특정아이디에게 접근권한 부여
mysql -e "flush privileges"  
~~~
   

### PHP,PHP-FPM와 Nginx연동
[ 1 ] nginx 설정파일 수정1
~~~
$ vi /etc/nginx/nginx.conf
~~~
~~~
user  nginx;
worker_processes  auto;
error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;
events {
    worker_connections  1024;
}
http {
    include  /etc/nginx/mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile        on;
    keepalive_timeout  65;
    include /etc/nginx/conf.d/*.conf;
}
~~~

[ 2 ] nginx 설정파일 수정2
~~~
$ vi /etc/nginx/conf.d/default.conf
~~~
~~~
server {
    listen 80;
    server_name 프로젝트명;
	root /vagrant/www/public;
	index index.php index.html index.htm;
	location ~ \.php$ {
            try_files $uri /index.php =404;
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
        }
    location / {
        try_files $uri $uri/ /index.php?$query_string;
        proxy_set_header REMOTE_ADDR $remote_addr;
        proxy_set_header X-Real-IP   $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header ORIGIN "";
        proxy_set_header Host $http_host;
        proxy_set_header X-Forwarded-Host $host;
    }
}
~~~

[ 3 ] php-fpm 설정파일 수정
~~~
$ vi /etc/php-fpm.d/www.conf
~~~

listen = 127.0.0.1:9000 으로 되어 있는 부분을 listen = /var/run/php-fpm/php-fpm.sock로 변경 <br>
~~~
;listen.owner = nobody 부분을 listen.owner = nginx로 변경
;listen.group = nobody 부분을 listen.group = nginx로 변경
;listen.mode = 0660 부분을 주석해제

listen = /var/run/php-fpm/php-fpm.sock
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
~~~


### php.ini 날짜 지역 설정 / laravel 날짜 지역 설정 
~~~
$ vi /etc/php.ini
~~~
~~~
$ :/date.timezone
~~~
~~~
$ date.timezone = Asia/Seoul 로 수정
~~~
라라벨의 경우에 config/app.php에 'timezone' => 'Asia/Seoul',로 설정


### redis 설치 및 설정
[ 1 ] 레디스 설치
~~~
$ yum install -y redis		
~~~
   
[ 2 ] 레디스 상태 확인 / 레디스 시작 / 레디스 자동시작
~~~
systemctl status redis
systemctl start redis
systemctl enable redis
~~~

[ 3 ] 레디스 커멘드 접속
~~~
$ redis-cli
~~~

[ 4 ] 레디스 명령어들
~~~
keys * #키 조회
keys *키* #특정 키 조회
set 키 값 #레디스 저장
get 키 #레디스 가져오기
~~~



 



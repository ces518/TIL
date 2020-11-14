# Centos Nginx 설치 및 세팅하기

### Yum 저장소에 Nginx 추가
- yum 기본 저장소에 nginx 가 없기 때문에 **외부 저장소를 추가** 해주어야 함
- /etc/yum.repos.d/ 디렉터리에 repo 관련 파일들 존재

```shell script
CentOS-Base.repo CentOS-CR.repo CentOS-Debuginfo.repo CentOS-Media.repo CentOS-Sources.repo CentOS-Vault.repo CentOS-fasttrack.repo microsoft-prod.repo
```

- /etc/yum.repos.d/nginx.repo 파일 추가후 외부 저장소 추가 설정

`nginx.repo`
```shell script
[nginx] 
name=nginx repo 
baseurl=http://nginx.org/packages/centos/7/$basearch/ 
gpgcheck=0 
enabled=1
```

> OS 별도 버전이 존재하므로 공식 페이지 참조

### Yum 으로 Nginx 설치
```shell script
yum install -y nginx
```

### 방화벽 포트 개방
- CentOS 7버전 기준 방화벽은 firewalld 를 사용한다.
- 기본적으로는 설치가 되어있지만 설치가 되어있지 않는 경우도 종종 존재함

#### 방화벽 설치 및 세팅
`firewalld 설치`
```shell script
yum install firewalld
```

`서버 부팅/재부팅시 자동으로 데몬으로 방화벽 실행`
```shell script
systemctl enable firewalld
systemctl start firewalld
```

`http 에 대한 방화벽 허용`
```shell script
firewall-cmd --permanent --add-service=http 
firewall-cmd --permanent --add-service=https
```

- 그외 설정 참조
    - https://uxgjs.tistory.com/162
    

### Nginx 데몬 실행
```shell script
systemctl start nginx
systemctl enable nginx
```

- 참조
    - https://holjjack.tistory.com/114
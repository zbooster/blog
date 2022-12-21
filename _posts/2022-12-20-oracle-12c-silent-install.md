---
layout: post
title: "Oracle 12c Silent 설치"
subtitle: "Oracle 12c GUI 없이 설치하기"
date: 2022-12-20 15:00:00 +0900
---
SW엔지니어들이 테스트 환경을 구축할때 GUI를 사용하지 않고 설치를 진행하는 것은 GUI를 사용하는 것보다 Resource나 소요시간을 매우 많이 줄일수 있기 때문에 선호하게 된다. 하지만, 인터넷의 Oracle 데이터베이스 설치과정을 설명하는 글은 대부분 실제환경에서 잘 사용하지 않는 Windows에서 GUI 설치를 주로 설명하기 때문에 이 글을 작성하게 되었다.


# 준비사항
Oracle Linux VM에 설치를 진행할 예정이다. 준비과정은 어렵지 않으나, 내용이 길기 때문에 아래 링크한 블로그내 다른 글을 참고하기 바란다.

참고: [Oracle Linux 7.9 설치](https://zbooster.github.io/blog/2022/12/20/oracle-linux-79-install.html)

# Oracle 12cR2 설치 전 준비단계
이 과정을 손쉽게 진행하기 위해 Oracle Linux를 선택했다. 
아래의 yum 명령을 수행하면 필요한 package 설치부터 Kernel 파라미터 수정, oracle 유저까지 만들어준다.
```bash
yum install -y oracle-database-server-12cR2-preinstall
```

설치과정이 끝나면 oracle 유저가 생성되었다. 해당 유저의 password를 설정해준다.
```bash
passwd oracle
```
Oracle을 설치할 디렉토리를 만들고 권한을 바꾸어 준다.
```bash
mkdir -p /u01/app/oracle/product/12.2.0.1
chown -R oracle:oinstall /u01
chmod -R 775 /u01
```

Oracle 환경변수 설정을 위해 oracle 유저로 전환하여 .bash_profile을 수정한다.
```bash
su - oracle
vi .bash_profile
```

.bash_profile 내부의 "User specific environment and startup programs"이후 모든 라인을 삭제하고 아래 내용을 붙여 넣는다.
```env
# User specific environment and startup programs
export ORACLE_HOME=/u01/app/oracle/product/12.2.0.1
export ORACLE_BASE=/u01/app/oracle
export ORACLE_SID=prod

export NLS_LANG=american_america.al32utf8
export NLS_DATE_FORMAT="yyyy-mm-dd:hh24:mi:ss"

PATH=$PATH:$HOME/.local/bin:$ORACLE_HOME/bin

export PATH
```
수정 완료한 프로파일을 수행하여 환경변수를 적용한다.
```bash
. .bash_profile
```

# Oracle 12cR2 설치
Oracle 홈페이지에 다운로드 받은 제품은 /tmp 위치에 올려 놓았다. 제품을 ORACLE_HOME에 풀고 설치를 진행한다.
(현재 Oracle 홈페이지에서 12c 제품을 바로 다운로드 할 수 없다. 다운로드 할 수 있는 방법은 이 [링크](https://velog.io/@composite/2020%EB%85%84-%ED%95%98%EB%B0%98%EA%B8%B0-%EC%9D%B4%ED%9B%84-Oracle-12c-%EB%8B%A4%EC%9A%B4%EB%A1%9C%EB%93%9C-%EB%B0%9B%EB%8A%94-%EB%B0%A9%EB%B2%95)를 참고하기 바란다.)

```bash
cd $ORACLE_HOME

unzip -qo /tmp/linuxx64_12201_database.zip

cd $ORACLE_HOME/database

./runInstaller -ignoreSysPrereqs -showProgress -silent       \
oracle.install.option=INSTALL_DB_SWONLY                      \
ORACLE_HOSTNAME=${HOSTNAME}                                  \
UNIX_GROUP_NAME=oinstall                                     \
INVENTORY_LOCATION=/u01/app/oraInventory                     \
SELECTED_LANGUAGES=en,en_GB                                  \
ORACLE_HOME=${ORACLE_HOME}                                   \
ORACLE_BASE=${ORACLE_BASE}                                   \
oracle.install.db.InstallEdition=EE                          \
oracle.install.db.OSDBA_GROUP=dba                            \
oracle.install.db.OSOPER_GROUP=dba                           \
oracle.install.db.OSBACKUPDBA_GROUP=dba                      \
oracle.install.db.OSDGDBA_GROUP=dba                          \
oracle.install.db.OSKMDBA_GROUP=dba                          \
oracle.install.db.OSRACDBA_GROUP=dba                         \
SECURITY_UPDATES_VIA_MYORACLESUPPORT=false                   \
DECLINE_SECURITY_UPDATES=true                                \
oracle.installer.autoupdates.option=SKIP_UPDATES
```
![Oracle 설치 완료](https://user-images.githubusercontent.com/100823210/208597390-aaec8019-e764-4892-a9a7-6aa83a416a1e.png){: width="100%" height="100%"}

설치가 정상적으로 수행되었다면 위와 같은 화면이 뜬다. exit하여 다시 root유저로 돌아간 다음 화면에 나온 스크립트를 수행한다.

```bash
exit
```

```bash
/u01/app/oraInventory/orainstRoot.sh
/u01/app/oracle/product/12.2.0.1/root.sh
```
이로써 Oracle Database의 엔진설치는 완료되었다. 

# 데이터베이스 생성
GUI에서는 데이터베이스 생성도 같이 진행되지만 silent 모드에서는 따로 진행을 해주어야 한다. 

명령 수행을 위해 oracle 유저로 전환한다.
```bash
su - oracle
```

아래 dbca 명령을 이용하여 데이터베이스를 생성한다.
```bash
dbca -silent -createDatabase                   \
-templateName General_Purpose.dbc              \
-gdbName ${ORACLE_SID}                         \
-sid ${ORACLE_SID}                             \
-createAsContainerDatabase false               \
-emConfiguration NONE                          \
-datafileDestination /u01/db_files             \
-storageType FS                                \
-characterSet AL32UTF8                         \
-totalMemory 2048                              \
-recoveryAreaDestination /u01/FRA              \
-sampleSchema true
```
명령을 수행하면 아래와 같이 SYS와 SYSTEM의 비밀번호를 어떻게 설정할것인지 물어본다. 너무 간단하게 만들면 화면과 같이 경고(Warning)문구가 뜬다. 필자는 테스트를 위한것이라서 기억하기 쉽게 지정했다.
![setup password](https://user-images.githubusercontent.com/100823210/208598713-d878c0ff-d44c-406f-921a-9e3c5006169b.png){: width="100%" height="100%"}

설치가 완료되면 아래 화면과 같은 메시지를 확인할 수 있다.
![creation complete](https://user-images.githubusercontent.com/100823210/208600147-8ba04580-8f90-4a8c-9791-2718197dc18c.png){: width="100%" height="100%"}

# 레퍼런스
- [Oracle 12c Silent Installation on Linux](https://www.support.dbagenesis.com/post/oracle-12c-silent-installation#viewer-fumdl)
- [2020년 하반기 이후 Oracle 12c 다운로드 받는 방법](https://velog.io/@composite/2020%EB%85%84-%ED%95%98%EB%B0%98%EA%B8%B0-%EC%9D%B4%ED%9B%84-Oracle-12c-%EB%8B%A4%EC%9A%B4%EB%A1%9C%EB%93%9C-%EB%B0%9B%EB%8A%94-%EB%B0%A9%EB%B2%95)

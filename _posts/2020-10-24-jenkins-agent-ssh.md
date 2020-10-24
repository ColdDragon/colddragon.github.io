# Jenkins agent ssh(Windows Master, Windows agent, WSL2 agent)

작성일시: 2020년 10월 24일 오전 10:14
작성자: Andrew Lee
최종 편집일시: 2020년 10월 24일 오후 5:32

- agent에서 ssh 설정
    - agent 서버의 ssh 서비스가 구동되는 상태에서 master에서 연결할 수 있도록 진행
    - Windows 10의 Openssh 서비스를 이용([https://lucidmaj7.tistory.com/178](https://lucidmaj7.tistory.com/178))
        - Windows **설정**에서 **앱**으로 진입

            ![jenkins_ssh_res/Untitled.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled.png)

        - 앱에서 **선택적 기능**으로 진입

            ![jenkins_ssh_res/Untitled%201.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%201.png)

        - **기능 추가** 로 진입

            ![jenkins_ssh_res/Untitled%202.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%202.png)

        - 검색창에서 openssh입력하여 설치

            ![jenkins_ssh_res/Untitled%203.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%203.png)

    - sshd 서비스 실행
        - powershell을 관리자 권한으로 오픈

            ![jenkins_ssh_res/Untitled%204.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%204.png)

        - sshd 서비스를 자동실행으로 등록

            ![jenkins_ssh_res/Untitled%205.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%205.png)

            - C:\>Start-Service -Name sshd
            - C:\>Set-Service -Name sshd -StartupType 'Automatic'
    - public key인증 방식으로 변경([https://medium.com/beyond-the-windows-korean-edition/windows-10-네이티브-방식으로-ssh-서버-설정하기-64988d87349](https://medium.com/beyond-the-windows-korean-edition/windows-10-%EB%84%A4%EC%9D%B4%ED%8B%B0%EB%B8%8C-%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C-ssh-%EC%84%9C%EB%B2%84-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0-64988d87349))
        - powershell을 관리자 권한으로 실행한 상태에서 진행
        - notepad.exe $env:PROGRAMDATA\ssh\sshd_config
            - 아래 값들을 주석을 제거하고 변경
                - `PubkeyAuthentication yes`
                - `PasswordAuthentication no`
                - `PermitEmptyPasswords no`
            - $HOME\.ssh\authorized_keys 파일을 사용하는 방식으로 변경하기 위해 아래 내용을 주석 처리(#)
                - #Match Group administrators
                #AuthorizedKeysFile
                #**PROGRAMDATA**/ssh/administrators_authorized_keys
        - mkdir "$HOME\.ssh" : .ssh경로 생성
        - authorized_keys 파일 생성 및 권한 변경

            ```powershell
            $authorizedKeyFilePath = “$HOME\.ssh\authorized_keys”
            New-Item $authorizedKeyFilePath
            notepad.exe $authorizedKeyFilePath
            icacls.exe $authorizedKeyFilePath /remove “NT AUTHORITY\Authenticated Users”
            icacls.exe $authorizedKeyFilePath /inheritance:r
            Get-Acl “$env:ProgramData\ssh\ssh_host_dsa_key” | Set-Acl $authorizedKeyFilePath
            ```

        - jenkins agent 설정
            - Jenkins >> Jenkins 관리 >> 노드 관리 >> 신규 노드

                ![jenkins_ssh_res/Untitled%206.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%206.png)

            - 기본 설정

                ![jenkins_ssh_res/Untitled%207.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%207.png)

            - 고급 설정에서 port과 command 설정

                ![jenkins_ssh_res/Untitled%208.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%208.png)

                - Jave.exe를 Path환경 설정으로 등록
                - Prefix Start Agent Command
                    - Remote root directory 경로가 C드라이드(윈도우 설치 경로)인 경우는 필요가 없지만 다른 드라이브를 사용할 경우 powershell command를 직접 수정하여 등록
                    - powershell -Command "chcp 65001 ; cd **d:\Jenkins** ; java -jar remoting.jar -workDir **d:\Jenkins** -jar-cache **d:\Jenkins/remoting/jarCache**" ; exit 0 ; rem

    - WSL2에서 ssh설정(Ubuntu v18.04) - ([https://parksb.github.io/article/21.html](https://parksb.github.io/article/21.html), [https://m.blog.naver.com/seongjin0526/221778212779](https://m.blog.naver.com/seongjin0526/221778212779))
        - hyper-V를 통해 내부 포트가 지정되고 부팅시마다 포트가 변경되어 자동으로 포트 포워딩을 해야 함
        - sudo ssh-keygen -A : public key 생성
        - sudo vi /etc/ssh/sshd_config
            - **Port 22 → Port 2022 : Windows가 점유한 22포트와 충돌나기 때문에 2022로 변경**
        - sudo service ssh --full-restart : ssh 서비스 등록
        - powersell용 스크립트 파일을 생성하여 작업 스캐줄러에 등록
            - wslbridge.ps1 파일에 저장

                ```powershell
                $remoteport = bash.exe -c "ifconfig eth0 | grep 'inet '"
                $found = $remoteport -match '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}';

                if( $found ){
                  $remoteport = $matches[0];
                } else{
                  echo "The Script Exited, the ip address of WSL 2 cannot be found";
                  exit;
                }

                #[Ports]

                #All the ports you want to forward separated by coma
                $ports=@(22,2022,80,443,10000,3000,5000);

                #[Static ip]
                #You can change the addr to your ip config to listen to a specific address
                $addr='0.0.0.0';
                $ports_a = $ports -join ",";

                #Remove Firewall Exception Rules
                iex "Remove-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' ";

                #adding Exception Rules for inbound and outbound Rules
                iex "New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Outbound -LocalPort $ports_a -Action Allow -Protocol TCP";
                iex "New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Inbound -LocalPort $ports_a -Action Allow -Protocol TCP";

                for( $i = 0; $i -lt $ports.length; $i++ ){
                  $port = $ports[$i];
                  iex "netsh interface portproxy delete v4tov4 listenport=$port listenaddress=$addr";
                  iex "netsh interface portproxy add v4tov4 listenport=$port listenaddress=$addr connectport=$port connectaddress=$remoteport";
                }
                ```

            - 작업 스캐줄러 등록
                - - ExecutionPolicy Bypass C:\Users\{Username}\wslbridge.ps1

                    ![jenkins_ssh_res/Untitled%209.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%209.png)

        - jenkins agent 설정
            - 기본 설정

                ![jenkins_ssh_res/Untitled%2010.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%2010.png)

            - 고급 설정

                ![jenkins_ssh_res/Untitled%2011.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%2011.png)

- master에서 ssh 설정(비대칭 암호키 사용)
    - agent 서버에 public key로 연결만 가능하면 되기에 sshd 서비스를 구동할 필요는 없다.
    - 비대칭 암호키 생성(마스터의 public key를 agent들에 배포)
        - ssh-keygen -t rsa : 완료할 때까지 엔터(기본값)
        - C:\Users\{Username}\.ssh\ 폴더상에 id_rsa, id_rsa.pub 파일 생성
        - scp $HOME/.ssh/id_rsa.pub {userid}@{ip address}:id_rsa.pub 를 통해 agent에 전송
        - cat $HOME/id_rsa.pub >> $HOME/.ssh/authorized_keys : public key를 agent의 authorized_keys 에 등록
        - ssh {userid}@{ip address} : agent 접속
        - linux환경인 경우 권한 설정
            - chmod 700 ~./ssh
            chmod 600 ~./ssh/id_rsa
            chmod 644 ~./ssh/id_ras.pub
            chmod 644 ~./ssh/authorized_keys
            chmod 644 ~./ssh/known_hosts
    - jenkins credentials 추가
        - Jenkins >> Jenkins 관리 >> Manage Credentials >> (global) >> Add Credentials

            ![jenkins_ssh_res/Untitled%2012.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%2012.png)

        - 아까 생성한 비밀키를 입력(C:\Users\{Username}\.ssh\id_rsa)

            ![jenkins_ssh_res/Untitled%2013.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%2013.png)

# Jenkins agent ssh(Windows Master, Windows agent, WSL2 agent)

작성일시: 2020년 10월 24일 오전 10:14
작성자: Andrew Lee
최종 편집일시: 2020년 10월 24일 오후 5:32

- agent에서 ssh 설정
    - agent 서버의 ssh 서비스가 구동되는 상태에서 master에서 연결할 수 있도록 진행
    - [Windows 10의 Openssh 서비스를 이용](https://lucidmaj7.tistory.com/178)
        - Windows **설정**에서 **앱**으로 진입

            ![jenkins_ssh_res/Untitled.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled.png){: width="100%" height="100%"}

        - 앱에서 **선택적 기능**으로 진입

            ![jenkins_ssh_res/Untitled%201.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%201.png){: width="100%" height="100%"}

        - **기능 추가** 로 진입

            ![jenkins_ssh_res/Untitled%202.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%202.png){: width="100%" height="100%"}

        - 검색창에서 openssh입력하여 설치

            ![jenkins_ssh_res/Untitled%203.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%203.png){: width="100%" height="100%"}

    - sshd 서비스 실행
        - powershell을 관리자 권한으로 오픈

            ![jenkins_ssh_res/Untitled%204.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%204.png){: width="100%" height="100%"}

        - sshd 서비스를 자동실행으로 등록

            ![jenkins_ssh_res/Untitled%205.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%205.png){: width="100%" height="100%"}

            ```
            C:\>Start-Service -Name sshd
            C:\>Set-Service -Name sshd -StartupType 'Automatic'
            ```
    - [public key인증 방식으로 변경](https://medium.com/beyond-the-windows-korean-edition/windows-10-%EB%84%A4%EC%9D%B4%ED%8B%B0%EB%B8%8C-%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C-ssh-%EC%84%9C%EB%B2%84-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0-64988d87349)
        - powershell을 관리자 권한으로 실행한 상태에서 진행
        - `notepad.exe $env:PROGRAMDATA\ssh\sshd_config`
            - 아래 값들의 주석을 제거하고 변경
                ```
                PubkeyAuthentication yes
                PasswordAuthentication no
                PermitEmptyPasswords no
                ```
            - $HOME\.ssh\authorized_keys 파일을 사용하는 방식으로 변경하기 위해 아래 내용을 주석 처리(#)
                ```
                #Match Group administrators
                #AuthorizedKeysFile **PROGRAMDATA**/ssh/administrators_authorized_keys
                ```
            - $ALLUSERSPROFILE\\ssh\\administrators_authorized_keys 권한 변경
                ```
                $authorizedKeyFilePath = "$env:ALLUSERSPROFILE\\ssh\\administrators_authorized_keys"
                New-Item $authorizedKeyFilePath
                notepad.exe $authorizedKeyFilePath
                icacls.exe $authorizedKeyFilePath /remove "NT AUTHORITY\\Authenticated Users"
                icacls.exe $authorizedKeyFilePath /inheritance:r
                Get-Acl "$env:ProgramData\\ssh\\ssh_host_dsa_key" | Set-Acl $authorizedKeyFilePath
                ```
        - .ssh경로 생성, authorized_keys 파일 생성 및 권한 변경

            ```powershell
            mkdir "$HOME\.ssh"
            $authorizedKeyFilePath = "$HOME\\.ssh\\authorized_keys"
            New-Item $authorizedKeyFilePath
            notepad.exe $authorizedKeyFilePath
            icacls.exe $authorizedKeyFilePath /remove "NT AUTHORITY\\Authenticated Users"
            icacls.exe $authorizedKeyFilePath /inheritance:r
            Get-Acl "$env:ProgramData\\ssh\\ssh_host_dsa_key" | Set-Acl $authorizedKeyFilePath
            ```

        - jenkins agent 설정
            - Jenkins >> Jenkins 관리 >> 노드 관리 >> 신규 노드

                ![jenkins_ssh_res/Untitled%206.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%206.png){: width="100%" height="100%"}

            - 기본 설정

                ![jenkins_ssh_res/Untitled%207.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%207.png){: width="100%" height="100%"}

            - 고급 설정에서 port과 command 설정

                ![jenkins_ssh_res/Untitled%208.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%208.png){: width="100%" height="100%"}

                - Jave.exe를 Path환경 설정으로 등록
                - Prefix Start Agent Command
                    - Remote root directory 경로가 C드라이드(윈도우 설치 경로)인 경우는 필요가 없지만 다른 드라이브를 사용할 경우 powershell command를 직접 수정하여 등록
                    - `powershell -Command "chcp 65001 ; cd **d:\Jenkins** ; java -jar remoting.jar -workDir **d:\Jenkins** -jar-cache **d:\Jenkins/remoting/jarCache**" ; exit 0 ; rem`

    - WSL2에서 ssh설정(Ubuntu v18.04)
        - [WSL2 설치](https://www.44bits.io/ko/post/wsl2-install-and-basic-usage)
            - `dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart`
            - `dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart`
            - [Updating the WSL 2 Linux Kernal | Microsoft Docs](https://docs.microsoft.com/ko-kr/windows/wsl/install-win10#step-4---download-the-linux-kernel-update-package)
            - Microsoft Store에서 Ubuntu 18.04 LTS설치                
            - wsl 설치후 기본 작업들
                - `Enter new UNIX username: jenkins_agent`
                - `Enter new UNIX password:`
                - `Retype new UNIX password:`
                - `sudo vi /etc/sudoers`
                    - `jenkins_agent ALL=(ALL:ALL) NOPASSWD: ALL`
                - `sudo apt update`
                - `sudo apt upgrade`
                - `sudo apt-get install openjdk-8-jdk`
                - `sudo apt install net-tools`
        - [WSL에서 ssh설치](https://www.tuwlab.com/ece/29302)
            - `sudo apt-get purge openssh-server`
            - `sudo apt-get install openssh-server`
            - `sudo nano /etc/ssh/sshd_config`
                ```
                Port 2022
                PasswordAuthentication yes
                PubkeyAuthentication no
                PermitEmptyPasswords no
                ```
            - `sudo service ssh --full-restart`
            - `sudo service ssh restart`
        - hyper-V를 통해 내부 포트가 지정되고 부팅시마다 포트가 변경되어 자동으로 포트 포워딩을 해야 함
        - `sudo vi /etc/ssh/sshd_config`
            - `Port 2022` : Windows가 점유한 22포트와 충돌나기 때문에 2022로 변경
            - `PubkeyAuthentication yes`
        - `sudo service ssh --full-restart : ssh 서비스 등록`
        - `sudo apt install net-tools : ifconfig 관련 `
        - master에서 id_rsa.pub가져오기
            - `mkdir ~/.ssh`
            - `chmod 700 ~/.ssh`
            - `scp {USER}@ipaddress:.ssh/id_rsa.pub ~/.ssh/authorized_keys`
            - `chmod 644 ~/.ssh/authorized_keys`
        - powersell용 스크립트 파일을 생성하여 작업 스캐줄러에 등록
            - [wslbridge.ps1](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/wslbridge.ps1) 파일에 저장

                ```powershell
                C:\Windows\System32\bash.exe -c "sudo service ssh start"
                $remoteport = C:\Windows\System32\bash.exe -c "ifconfig eth0 | grep 'inet '"
                $found = $remoteport -match '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}';

                if( $found ){
                  $remoteport = $matches[0];
                } else{
                  echo "The Script Exited, the ip address of WSL 2 cannot be found";
                  exit;
                }

                #[Ports]

                #All the ports you want to forward separated by coma
                $ports=@(2022);


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
                - 동작 속성에서 프로그래/스크립트
                    - powershell.exe의 전체 경로로 설정 : C:\Windows\system32\WindowsPowerShell\v1.0\powershell.exe
                - `-ExecutionPolicy Bypass {복사한 전체 경로}\wslbridge.ps1`

                    ![jenkins_ssh_res/Untitled%209.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%209.png){: width="100%" height="100%"}
                - 로그인한 사용자로 설정하고 암호를 입력해야함
                    ![jenkins_ssh_res/Untitled%209.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/jobscheduler.png){: width="100%" height="100%"}
                - [일괄 작업으로 로그온 설정](https://urban1980.tistory.com/42#:~:text=%EC%9D%BC%EB%8B%A8%20%EA%B4%80%EB%A6%AC%EB%8F%84%EA%B5%AC%20%3E%20%EB%A1%9C%EC%BB%AC%EB%B3%B4%EC%95%88,%EC%9D%84%20%ED%9A%8D%EB%93%9D%ED%95%A0%20%EC%88%98%20%EC%9E%88%EC%8A%B5%EB%8B%88%EB%8B%A4)
                - [서비스로 로그온 설정](https://help.tableau.com/current/server/ko-kr/runas_security.htm)

        - jenkins agent 설정
            - 기본 설정

                ![jenkins_ssh_res/Untitled%2010.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%2010.png){: width="100%" height="100%"}

            - 고급 설정

                ![jenkins_ssh_res/Untitled%2011.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%2011.png){: width="100%" height="100%"}

- master에서 ssh 설정(비대칭 암호키 사용)
    - agent 서버에 public key로 연결만 가능하면 되기에 sshd 서비스를 구동할 필요는 없다.
    - 비대칭 암호키 생성(마스터의 public key를 agent들에 배포)
        - `ssh-keygen -t rsa : 완료할 때까지 엔터(기본값)`
        - `C:\Users\{Username}\.ssh\ 폴더상에 id_rsa, id_rsa.pub 파일 생성`
        - `scp $HOME/.ssh/id_rsa.pub {userid}@{ip address}:id_rsa.pub 를 통해 agent에 전송`
        - `cat $HOME/id_rsa.pub >> $HOME/.ssh/authorized_keys : public key를 agent의 authorized_keys 에 등록`
        - `ssh {userid}@{ip address} : agent 접속`
        - linux환경인 경우 권한 설정
            ```
            chmod 700 ~/.ssh
            chmod 600 ~/.ssh/id_rsa
            chmod 644 ~/.ssh/id_rsa.pub
            chmod 644 ~/.ssh/authorized_keys
            chmod 644 ~/.ssh/known_hosts
            ```
    - jenkins credentials 추가
        - Jenkins >> Jenkins 관리 >> Manage Credentials >> (global) >> Add Credentials

            ![jenkins_ssh_res/Untitled%2012.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%2012.png){: width="100%" height="100%"}

        - 아까 생성한 비밀키를 입력(C:\Users\{Username}\.ssh\id_rsa)

            ![jenkins_ssh_res/Untitled%2013.png](https://raw.githubusercontent.com/ColdDragon/colddragon.github.io/master/_posts/jenkins_ssh_res/Untitled%2013.png){: width="100%" height="100%"}

# Jenkins agent ssh

작성일시: 2020년 10월 24일 오전 10:14
작성자: Andrew Lee
최종 편집일시: 2020년 10월 24일 오전 10:54

Jenkins agent연동을 ssh방식으로 진행

- master에서 ssh 설정(비대칭 암호키 사용)
    - Windows 10의 Openssh 서비스를 이용하여 master를 구축(Windows 10 1809, Windows server 2019이상의 버전에서 OpenSSH 서버와 클라이언트를 지원)
        - Windows **설정**에서 **앱**으로 진입

            ![Jenkins%20agent%20ssh%206b0e3e4959464e1eafc4a0be0fffd6a7/Untitled.png](Jenkins%20agent%20ssh%206b0e3e4959464e1eafc4a0be0fffd6a7/Untitled.png)

        - 앱에서 **선택적 기능**으로 진입

            ![Jenkins%20agent%20ssh%206b0e3e4959464e1eafc4a0be0fffd6a7/Untitled%201.png](Jenkins%20agent%20ssh%206b0e3e4959464e1eafc4a0be0fffd6a7/Untitled%201.png)

        - **기능 추가** 로 진입

            ![Jenkins%20agent%20ssh%206b0e3e4959464e1eafc4a0be0fffd6a7/Untitled%202.png](Jenkins%20agent%20ssh%206b0e3e4959464e1eafc4a0be0fffd6a7/Untitled%202.png)

        - 검색창에서 openssh입력하여 설치

            ![Jenkins%20agent%20ssh%206b0e3e4959464e1eafc4a0be0fffd6a7/Untitled%203.png](Jenkins%20agent%20ssh%206b0e3e4959464e1eafc4a0be0fffd6a7/Untitled%203.png)

        - sshd 서비스 실행
            - powershell을 관리자 권한으로 오픈

                ![Jenkins%20agent%20ssh%206b0e3e4959464e1eafc4a0be0fffd6a7/Untitled%204.png](Jenkins%20agent%20ssh%206b0e3e4959464e1eafc4a0be0fffd6a7/Untitled%204.png)

            - sshd 서비스를 자동실행으로 등록

                ![Jenkins%20agent%20ssh%206b0e3e4959464e1eafc4a0be0fffd6a7/Untitled%205.png](Jenkins%20agent%20ssh%206b0e3e4959464e1eafc4a0be0fffd6a7/Untitled%205.png)

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

                ![Jenkins%20agent%20ssh%206b0e3e4959464e1eafc4a0be0fffd6a7/Untitled%206.png](Jenkins%20agent%20ssh%206b0e3e4959464e1eafc4a0be0fffd6a7/Untitled%206.png)

                ![Jenkins%20agent%20ssh%206b0e3e4959464e1eafc4a0be0fffd6a7/Untitled%207.png](Jenkins%20agent%20ssh%206b0e3e4959464e1eafc4a0be0fffd6a7/Untitled%207.png)

            - 아까 생성한 비밀키를 입력(C:\Users\{Username}\.ssh\id_rsa)

                ![Jenkins%20agent%20ssh%206b0e3e4959464e1eafc4a0be0fffd6a7/Untitled%208.png](Jenkins%20agent%20ssh%206b0e3e4959464e1eafc4a0be0fffd6a7/Untitled%208.png)

            -

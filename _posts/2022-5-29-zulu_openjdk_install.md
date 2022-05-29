## azul zulu openjdk 11 install
> Reference site
> - https://freedeveloper.tistory.com/200
> - https://blo-gu.tistory.com/7

- Windows 설치
https://www.azul.com/downloads/?version=java-11-lts&os=windows&architecture=x86-64-bit&package=jdk

- Linux ubuntu 설치
- 기존 설치된 jdk제거
> apt-get remove oracle*
> apt-get autoremove --purge
> apt-get autoclean

> apt-get install software-properties-common
> apt-add-repository 'deb http://repos.azulsystems.com/ubuntu stable main'
> apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 0xB1998361219BD9C9
> apt-get update && apt-cache search zulu
> apt-get install zulu-11
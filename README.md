# LinuxNetAdmin

Linux 네트워크 설정과 서버 서비스의 연결 관계를 정리한 학습 저장소입니다. NetworkManager를 통한 연결 상태 확인에서 시작해 DNS·웹·파일 공유 등 서비스 접근 문제를 네트워크, 프로세스, 방화벽, SELinux 순서로 살펴봅니다.

실습 환경의 흔적으로 남아 있는 파일은 셸·Vim 설정과 `fstab` 기록입니다. DNS, 웹, 메일, 파일 공유 등의 실제 서버 설정 파일과 실행 결과는 포함되어 있지 않으므로, 아래 서비스 항목은 학습 범위와 **확인 방법**을 설명합니다.

## 확인 흐름

| 단계 | 확인할 것 | 예시 |
| --- | --- | --- |
| 연결 | 인터페이스와 연결 프로필, 주소, 기본 경로 | `nmcli device status`, `nmcli connection show --active`, `ip address`, `ip route` |
| 이름 해석 | 대상 이름이 기대한 주소를 가리키는지 | `dig example.com` |
| 서비스 | 프로세스 상태와 수신 포트, 애플리케이션 응답 | `systemctl status sshd`, `ss -lntup`, 서비스별 클라이언트 |
| 접근 제어 | 적용된 firewalld 영역과 SELinux 상태 | `firewall-cmd --get-active-zones`, `sestatus` |

서비스가 응답하지 않을 때는 앞 단계의 결과를 확인한 뒤 해당 서비스의 로그와 설정을 대조합니다. 도메인·서비스 이름은 점검 대상에 맞게 바꿉니다. 명령은 대상 환경에 패키지와 권한이 갖춰져 있을 때 실행하는 **점검 예시**이며, 이 저장소의 서버가 동작함을 증명하는 결과가 아닙니다.

## 다룬 영역

- **네트워크:** NetworkManager 연결 프로필, 주소·라우팅·DNS 확인, 안전한 변경 순서, 본딩 개념 — [네트워크 점검](01_NET/README.md)
- **서비스:** DNS·DHCP, HTTP, FTP, 메일, NFS·SMB, SSH, 시간 동기화 및 그 밖의 학습 주제 — [서비스와 접근 제어](docs/services-and-access.md)
- **접근 제어:** firewalld 영역·NAT·포트 전달과 SELinux 비표준 포트 정책을 서비스 점검 흐름에 연결

학습노트의 예전 `ifcfg`·`iptables` 명령을 그대로 실행하기보다 현재 RHEL 9 계열에서 쓰는 연결 프로필과 방화벽 관리 방식을 따릅니다. 버전별 차이는 [네트워크 점검](01_NET/README.md#현재-환경에-적용할-때)과 [서비스와 접근 제어](docs/services-and-access.md#현재-환경에-적용할-때)에 간단히 정리했습니다.

## 저장소 파일

| 경로 | 내용과 해석 |
| --- | --- |
| [`01_NET/README.md`](01_NET/README.md) | 네트워크 연결 및 진단 절차 |
| [`docs/services-and-access.md`](docs/services-and-access.md) | 서비스별 확인 지점과 접근 제어 점검 |
| [`00_files/fstab`](00_files/fstab) | 특정 실습 시스템의 마운트 설정 기록. 주석의 LVM·RAID·SMB 항목은 설정이 적용된 상태를 뜻하지 않음 |
| [`00_env/bashrc.txt`](00_env/bashrc.txt) | 셸 환경 및 서비스별 이동·조회 별칭. 별칭의 존재가 서비스 구성을 입증하지는 않음 |
| [`00_env/vimrc.txt`](00_env/vimrc.txt) | Vim 편집 환경 |

`fstab`의 장치 식별자와 네트워크 공유 예시는 특정 환경에 종속됩니다. 다른 시스템의 `/etc/fstab`으로 복사하거나 `bashrc.txt`를 그대로 로드하지 말고, 실제 장치·경로와 별칭의 동작을 먼저 확인해야 합니다. 특히 `bashrc.txt`에는 삭제를 수행하는 별칭과 브라우저 격리를 해제하는 옵션이 있습니다.

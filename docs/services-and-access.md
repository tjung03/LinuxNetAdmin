# 서비스와 접근 제어 점검

학습 범위에는 DNS·DHCP·웹·FTP·메일·NFS·autofs·SMB·로그·SSH·시간 동기화·DB·iSCSI·컨테이너가 포함됩니다. 이 문서는 그중 네트워크 서비스 점검에 직접 연결되는 항목을 묶었습니다. 저장소에는 각 서비스의 설정, 테스트 로그, 운영 결과가 없으며 아래의 명령과 기대 지점은 **실습 환경에서 확인할 절차**입니다.

## 서비스별 확인 지점

| 영역 | 확인 질문 | 예시 점검 |
| --- | --- | --- |
| DNS | 조회한 이름의 레코드가 기대한 값인가? 서버에 질의할 수 있는가? | `dig @<DNS서버> <도메인>`; DNS는 UDP와 TCP 53을 사용 |
| DHCP | 클라이언트가 주소·게이트웨이·DNS를 받았는가? | 클라이언트에서 `ip address`, `ip route`, `nmcli device show` 확인 |
| HTTP(S) | 서버가 수신 중이고 URL 요청에 응답하는가? | `ss -lntup`; `curl -i '<URL>'`; HTTP 80/TCP, HTTPS 443/TCP는 통상 포트 |
| FTP·메일 | 서버 설정과 인증, 서비스 로그가 요청 흐름에 맞는가? | 서비스 상태와 로그를 확인한 뒤 해당 클라이언트로 실제 요청 테스트 |
| NFS·SMB | 내보낸 경로·공유 권한과 클라이언트의 마운트 상태가 맞는가? | 서버의 공유 설정과 클라이언트의 `findmnt`를 각각 확인 |
| SSH·시간 동기화 | 원격 연결 경로와 시간 소스가 정상인가? | SSH 접속 결과, `chronyc sources`를 별도 확인; NTP의 기본 포트는 123/UDP |

`<...>`는 대상 환경의 값으로 교체할 자리표시자입니다. `curl -i`는 응답 헤더와 본문을 요청하며, 출력의 HTTP 상태 코드도 서비스 동작과 별도로 해석해야 합니다. 포트는 서버 설정과 방화벽의 현재 규칙을 확인해야 확정할 수 있습니다. NFS는 버전과 관련 RPC 서비스에 따라 필요한 포트가 달라질 수 있습니다. `00_files/fstab`에는 주석이 아닌 로컬 마운트 설정 줄과 주석 처리된 SMB 마운트 예시가 있으나, 현재 마운트 상태나 SMB 공유 연결을 증명하지는 않습니다. `credentials=`는 자격 증명 **파일 경로** 옵션이며, 저장소에는 그 파일이 없습니다.

## 접근 실패를 좁히는 순서

1. 클라이언트의 주소·경로와 DNS 응답을 확인합니다. IP로는 접근되는데 이름으로 실패한다면 이름 해석 경로를 별도로 확인합니다.
2. 서버에서 해당 서비스의 상태, 수신 주소·포트, 요청 시점의 로그를 확인합니다. 수신 포트가 있어도 애플리케이션 응답이 올바르다는 뜻은 아닙니다.
3. `firewall-cmd --get-active-zones`로 인터페이스가 속한 영역을 찾고, `firewall-cmd --zone=<영역> --list-all`로 해당 영역의 **현재 적용 규칙**을 확인합니다. 영구 설정은 `--permanent`를 붙여 별도로 확인합니다.
4. `sestatus`로 SELinux 모드를 확인하고, 접근 거부가 의심되면 `ls -Z <경로>`와 `ausearch -m AVC,USER_AVC,SELINUX_ERR,USER_SELINUX_ERR -ts recent`로 레이블과 감사 기록을 살펴봅니다. 오류를 해결하기 전에 SELinux를 끄는 방법으로 원인을 단정하지 않습니다.
5. 권한, 파일 컨텍스트, 서비스 설정을 대조한 뒤 변경이 필요한 경우 대상 환경에서 적용하고 동일한 요청으로 다시 확인합니다. 이 저장소에는 이러한 변경을 적용한 기록이 없습니다.

기존 [`bashrc.txt`](../00_env/bashrc.txt)의 서비스 관련 별칭은 설정 파일로 이동하거나 로그를 열기 위한 편의 명령입니다. 실제 설정 파일이나 서비스가 이 저장소에 포함되어 있다는 뜻은 아닙니다.

## 현재 환경에 적용할 때

- 방화벽을 firewalld로 관리하는 환경에서는 연결된 영역과 `firewall-cmd` 규칙을 확인합니다. 학습노트의 단순 작업에는 `iptables`를 권한다는 문구를 현행 지침으로 사용하지 않습니다. RHEL 9에서는 `iptables-nft`가 사용 중단 예정이고 firewalld·nftables를 문서화합니다.
- 기존 셸 별칭의 `netstat` 대신 위 점검에서는 `ss`를 사용합니다. 이 별칭은 편의 기록이므로 최신 운영 절차로 그대로 복사할 필요가 없습니다.
- FTP 학습 내용은 서비스 이해의 기록입니다. 자격 증명과 파일 전송을 보호해야 하는 사용 사례에서는 SSH 기반 SFTP 등 암호화된 방식을 선택하고, SFTP와 FTPS를 동일한 프로토콜로 취급하지 않습니다.

참고: [Red Hat 방화벽 문서](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_firewalls_and_packet_filters/using-and-configuring-firewalld_firewall-packet-filters), [RHEL 9 사용 중단 항목](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/9.0_release_notes/deprecated_functionality), [SELinux 문제 해결](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/using_selinux/troubleshooting-problems-related-to-selinux_using-selinux), [DNS·DHCP](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html-single/managing_networking_infrastructure_services/managing_networking_infrastructure_services), [NFS·SMB](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_and_using_network_file_services/index), [시간 동기화](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_basic_system_settings/configuring-time-synchronization_configuring-basic-system-settings), [OpenSSH](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_basic_system_settings/assembly_using-secure-communications-between-two-systems-with-openssh_configuring-basic-system-settings)

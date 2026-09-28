# 네트워크 연결과 진단

NetworkManager는 네트워크 장치와 연결 프로필을 관리합니다. 프로필에 기록된 값과 현재 장치에 적용된 주소·경로를 구별하여 확인해야 합니다. 아래는 RHEL 계열 실습 환경을 기준으로 한 **조회 절차**입니다. 이 저장소에는 연결 프로필이나 실제 장치의 출력이 저장되어 있지 않습니다.

## 1. 연결 상태

```bash
nmcli device status
nmcli connection show
nmcli connection show --active
ip address
ip route
```

장치가 연결되어 있는지, 어떤 프로필이 활성화되어 있는지, 주소와 기본 경로가 예상한 네트워크에 맞는지 순서대로 읽습니다. 여러 인터페이스가 있다면 프로필 이름과 장치 이름을 혼동하지 않습니다. 주소나 게이트웨이를 변경할 때는 원격 접속이 끊길 수 있으므로 해당 호스트의 접속 경로를 확인한 뒤 별도로 작업합니다.

## 2. 이름 해석과 도달성

```bash
cat /etc/resolv.conf
nmcli device show
dig example.com
ip route get 1.1.1.1
```

예시 도메인·목적지는 점검할 환경의 값으로 바꿉니다. `dig` 결과는 DNS 응답과 조회한 이름을 함께 읽고, `ip route get`으로 목적지 경로를 확인합니다. `/etc/resolv.conf`가 로컬 DNS 스텁을 가리키는 구성에서는 이 파일만으로 업스트림 DNS 서버를 알 수 없으므로 연결별 DNS 정보도 확인합니다. ICMP를 추가로 확인하더라도 차단될 수 있으므로 `ping` 실패만으로 서비스 장애를 확정하지 않습니다. 서비스 접근은 해당 프로토콜의 클라이언트로 확인합니다.

## 3. 서비스까지 추적

```bash
ss -lntup
firewall-cmd --get-active-zones
sestatus
```

수신 주소·포트, 인터페이스에 연결된 방화벽 영역, SELinux 상태를 서로 대조합니다. 서비스 상태 확인, 요청 테스트와 로그 분석은 [서비스와 접근 제어](../docs/services-and-access.md)에 이어집니다. 본딩은 여러 인터페이스를 하나의 논리 연결로 다루는 학습 주제이지만, 이 저장소에는 본딩 설정이나 장애 전환 결과가 없습니다.

## 현재 환경에 적용할 때

RHEL 9에서 NetworkManager의 기본 연결 프로필 저장 형식은 keyfile(`.nmconnection`)입니다. 이전 `ifcfg` 형식은 호환을 위한 대상이며 현재는 사용 중단 예정 형식으로 분류됩니다. 이 문서의 조회에는 `nmcli`와 `ip`를 사용합니다. 실제 변경은 배포판 버전과 기존 프로필을 확인한 후 진행합니다.

참고: [Red Hat Enterprise Linux 9 네트워크 관리 문서](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html-single/configuring_and_managing_networking/index), [NetworkManager keyfile 형식](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_and_managing_networking/assembly_networkmanager-connection-profiles-in-keyfile-format_configuring-and-managing-networking)

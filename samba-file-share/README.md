# Linux Samba 파일 공유 프로젝트

Rocky Linux에서 Samba를 설치하여 Windows와 파일을 공유하는 실습 프로젝트입니다.

- 운영체제: Rocky Linux 10
- 공유 디렉터리: `/srv/samba/share`
- Samba 사용자: `sambauser`
- Windows 접속 주소: `\\192.168.0.15\share`

---

## 프로젝트 개요

Samba는 SMB(Server Message Block) 프로토콜을 사용하여 Linux와 Windows 간 파일 공유를 가능하게 하는 서비스입니다.

이번 실습에서는 다음 작업을 수행했습니다.

- Samba 및 smbclient 설치
- 공유 디렉터리 생성
- Linux 사용자 생성
- Samba 사용자 등록
- `/etc/samba/smb.conf` 설정
- SELinux 컨텍스트 설정
- 방화벽 설정
- Samba 서비스 실행 및 자동 시작 등록
- Windows 파일 탐색기에서 공유 폴더 접속

---

## 시스템 구성도

```text
┌───────────────────────────────┐
│           Windows PC          │
│                               │
│  파일 탐색기                  │
│  \\192.168.0.15\share         │
└───────────────┬───────────────┘
                │
                │ SMB (TCP 445)
                ▼
┌───────────────────────────────┐
│       Rocky Linux Server      │
│        192.168.0.15           │
│                               │
│  Samba 서비스 (smbd)          │
│  사용자: sambauser            │
│                               │
│  공유 디렉터리                │
│  /srv/samba/share             │
└───────────────────────────────┘
```

---

## 프로젝트 구조

```text
linux-samba-share/
├── README.md
├── smb.conf
└── screenshots/
    ├── systemctl-status-smb.png
    ├── smbclient-list.png
    └── windows-access-success.png
```

---

## 1. Samba 설치

```bash
dnf install -y samba samba-client
```

---

## 2. 공유 디렉터리 생성

```bash
mkdir -p /srv/samba/share
chmod 777 /srv/samba/share
```

> 학습용 실습에서는 권한을 단순하게 설정하기 위해 `777`을 사용했습니다.

---

## 3. Linux 사용자 생성

```bash
useradd sambauser
passwd sambauser
```

---

## 4. Samba 사용자 등록

```bash
smbpasswd -a sambauser
```

> Windows에서 접속할 때는 여기서 설정한 비밀번호를 사용합니다.

---

## 5. Samba 설정

`/etc/samba/smb.conf` 파일의 마지막에 다음 내용을 추가합니다.

```ini
[share]
    comment = Samba Share Directory
    path = /srv/samba/share
    browseable = yes
    read only = no
    writable = yes
    guest ok = no
    valid users = sambauser
    create mask = 0777
    directory mask = 0777
```

---

## 6. 설정 확인

```bash
testparm
```

정상 출력 예시:

```text
Loaded services file OK.
Server role: ROLE_STANDALONE
```

---

## 7. SELinux 설정

```bash
chcon -R -t samba_share_t /srv/samba/share
```

---

## 8. 방화벽 설정

```bash
firewall-cmd --permanent --add-service=samba
firewall-cmd --reload
firewall-cmd --list-services
```

출력 결과에 `samba`가 포함되어 있어야 합니다.

---

## 9. Samba 서비스 실행

```bash
systemctl enable smb
systemctl start smb
systemctl status smb
```

정상 상태 예시:

```text
Active: active (running)
```

---

## 10. 공유 목록 확인

```bash
smbclient -L localhost -U sambauser
```

정상 출력 예시:

```text
Sharename       Type      Comment
---------       ----      -------
share           Disk      Samba Share Directory
IPC$            IPC       IPC Service
sambauser       Disk      Home Directories
```

---

## 11. Windows에서 접속

Windows 파일 탐색기 주소창에 다음을 입력합니다.

```text
\\192.168.0.15\share
```

로그인 정보:

- 사용자명: `sambauser`
- 비밀번호: `smbpasswd -a sambauser`에서 설정한 비밀번호

---

## 문제 해결 과정

### 문제 1. Windows에서 접속 실패
- 원인: `[share]` 설정이 `smb.conf`에 저장되지 않음
- 해결: 설정 추가 후 `testparm`, `systemctl restart smb`

### 문제 2. `smbclient` 명령어 없음
- 원인: `samba-client` 패키지 미설치
- 해결: `dnf install -y samba-client`

### 문제 3. 공유 목록에 `share`가 나타나지 않음
- 원인: 설정 파일 미반영
- 해결: `smb.conf` 수정 후 서비스 재시작

---

## 자주 사용하는 명령어

```bash
systemctl status smb
systemctl restart smb
testparm
firewall-cmd --list-services
pdbedit -L
smbclient -L localhost -U sambauser
ls -ld /srv/samba/share
```

---

## 보안 참고 사항

이번 프로젝트는 학습 목적이므로 단순하게 구성했습니다.

실무에서는 다음과 같은 보안 정책을 적용합니다.

- `chmod 777` 사용 지양
- 최소 권한 원칙 적용
- 공인 IP에 SMB 직접 노출 금지
- VPN(예: WireGuard) 사용 권장

---

## 학습한 내용

- Linux 시스템 관리
- Samba 파일 공유
- SELinux 설정
- 방화벽 설정
- Windows 네트워크 공유
- 문제 해결 및 디버깅
- GitHub 문서화

---

## 참고 자료

- https://www.samba.org/
- https://rockylinux.org/
- https://docs.redhat.com/

---

## 작성자

- GitHub: https://github.com/YOUR_USERNAME

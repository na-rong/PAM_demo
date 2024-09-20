# 🔐 리눅스 PAM 비밀번호 정책 설정

- **PAM이란?**: Pluggable Authentication Modules
- **목적**: PAM을 사용하여 리눅스 시스템에서 비밀번호 정책을 설정합니다.

---

## 목차

1. ubuntu 생성하기
2. 고정 IP 설정하기
3. 비밀번호 최소 길이 8자리로 설정하기

---

## 🚀 설정 단계

### 1. myserver03 생성

기존에 있던 **myserver01**을 복제하여 **myserver03**을 생성합니다.
![image](https://github.com/user-attachments/assets/290d843e-b5d9-421b-95b3-4c16011aaebd)
- 서버 복제 후, 도구에서 포트 포워딩을 설정해 다른 서버와 포트가 겹치지 않도록 주의합니다.

---

### 2. 고정 IP 설정

Netplan 설정 파일을 편집하여 고정 IP를 설정합니다.

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

다음 내용을 추가하여 IP를 설정합니다.

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      addresses:
        - 10.0.2.21/24  # 고정 IP 주소
      routes:
        - to: default
          via: 10.0.2.1  # 게이트웨이
      nameservers:
        addresses:
          - 8.8.8.8
      dhcp4: false
```

설정 후 적용

```bash
sudo netplan apply
```

---

### 3. PAM 비밀번호 정책 설정

**PAM 비밀번호 설정** 파일을 수정하여 최소 비밀번호 길이를 8자리로 설정합니다.

```bash
sudo nano /etc/pam.d/common-password
```

다음 줄이 포함되어 있는지 확인하거나, 없다면 추가합니다.

```bash
password requisite pam_pwquality.so retry=3 minlen=8
```

- **`retry=3`**: 비밀번호를 3번까지 시도할 수 있도록 설정
- **`minlen=8`**: 비밀번호 최소 길이 8자리 설정

### 4. PAM 패키지 설치

PAM 패키지를 설치합니다.

```bash
sudo apt install libpam-pwquality
```

---

### 5. 서비스 재시작 및 설정 확인

**systemd-logind** 서비스를 재시작합니다.

```bash
sudo systemctl restart systemd-logind
```

설정 파일에 **pam_pwquality.so** 모듈이 포함되어 있는지 확인합니다.

```bash
sudo cat /etc/pam.d/common-password | grep pam_pwquality
```

정상 출력 예시

```bash
password        requisite               pam_pwquality.so retry=3 minlen=8
```

---

### 6. 결과 확인

설정을 완료한 후, **test**로 비밀번호를 설정하려고 시도하면 8자리 이상을 요구하는 것을 확인할 수 있습니다.
![image](https://github.com/user-attachments/assets/6cdb1c5a-6aa6-4bfa-9b42-5da39fcb300c)


---


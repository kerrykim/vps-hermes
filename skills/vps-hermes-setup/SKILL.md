---
name: vps-hermes-setup
description: VPS에 Hermes Agent를 설치하는 전체 셋업을 단계별로 진행합니다. 로컬 PC Tailscale 설치부터 VPS 유저 생성, Tailnet 구성, SSH 키 설정, Hermes 설치까지 전 과정을 커버합니다. 사용자가 "VPS 설정", "hermes 설치", "tailscale 설정", "vps 셋업" 등을 언급할 때 사용하세요.
---

# VPS Hermes Setup

VPS에 Hermes Agent를 설치하는 전체 과정을 단계별로 진행합니다.
로컬 PC Tailscale 설치 → VPS 유저 생성 → Tailnet 구성 → SSH 설정 → Hermes 설치 순서로 진행합니다.

## 사전 준비

시작 전에 다음 정보를 사용자에게 요청합니다:
1. VPS IP 주소
2. 생성할 유저 이름 (예: `deploy`, `ubuntu` 등)
3. 유저 비밀번호
4. SSH 접속에 사용할 VPS 별칭 (예: `myserver` → `ssh myserver` 로 접속)

## 단계 1: 로컬 PC에 Tailscale 설치

현재 OS를 확인하고 적절한 방법으로 설치합니다.

```bash
# OS 확인
uname -s
```

### Linux
```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

### macOS
```bash
brew install tailscale
sudo tailscale up
```

설치 후 출력되는 auth URL을 사용자에게 안내하고 브라우저에서 로그인하도록 합니다.
로그인 완료 후 tailscale 상태를 확인합니다:
```bash
tailscale status
```

**중단점:** 사용자가 Tailscale 로그인을 완료했는지 확인 후 다음 단계로 진행합니다.

## 단계 2: VPS 유저 생성 (root로 접속)

VPS에 root로 SSH 접속하여 유저를 생성합니다.

```bash
ssh root@<VPS_IP>
```

접속 후 유저 생성:
```bash
# 유저 생성
useradd -m -s /bin/bash <USERNAME>

# 비밀번호 설정
echo "<USERNAME>:<PASSWORD>" | chpasswd

# sudo 권한 부여
usermod -aG sudo <USERNAME>

# 설정 확인
id <USERNAME>
```

**주의:** 비밀번호는 화면에 출력되지 않도록 처리합니다.
완료 후 root SSH 세션을 종료합니다.

## 단계 3: VPS에 Tailscale 설치 (유저 계정으로)

유저 계정으로 VPS에 SSH 접속합니다:
```bash
ssh <USERNAME>@<VPS_IP>
```

Tailscale 설치:
```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

출력된 auth URL을 사용자에게 보여주고 브라우저에서 VPS를 Tailnet에 승인하도록 안내합니다.

**중단점:** 사용자가 VPS Tailscale 승인을 완료했는지 확인합니다.

승인 후 Tailscale IP 확인:
```bash
tailscale ip -4
```

이 IP를 저장해 둡니다 (SSH config에 사용).

## 단계 4: SSH 키 설정 (로컬 PC에서)

로컬 PC에서 SSH 키가 없으면 생성합니다:
```bash
# 키 존재 여부 확인
ls ~/.ssh/id_*.pub 2>/dev/null || ssh-keygen -t ed25519 -C "$(whoami)@$(hostname)" -N "" -f ~/.ssh/id_ed25519
```

VPS에 공개키를 복사합니다:
```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub <USERNAME>@<VPS_IP>
```

`~/.ssh/config`에 VPS 별칭을 등록합니다:
```bash
cat >> ~/.ssh/config << EOF

Host <VPS_ALIAS>
    HostName <TAILSCALE_IP>
    User <USERNAME>
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60
EOF
chmod 600 ~/.ssh/config
```

접속 테스트:
```bash
ssh <VPS_ALIAS>
```

`ssh <VPS_ALIAS>` 로 접속이 되면 성공입니다.

## 단계 5: Hermes 설치

VPS에 SSH 접속 후 Hermes를 설치합니다:
```bash
ssh <VPS_ALIAS>
```

Hermes 설치:
```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
source ~/.bashrc
hermes --version
```

설치 완료 후 초기 설정:
```bash
hermes setup
```

## 완료 확인

다음 항목을 확인합니다:
- [ ] 로컬 PC에서 `tailscale status` 로 로컬 PC와 VPS 모두 보임
- [ ] `ssh <VPS_ALIAS>` 로 비밀번호 없이 접속 가능
- [ ] VPS에서 `hermes` 명령 실행 가능

## 문제 해결

### SSH 접속 실패
```bash
# 권한 확인
ssh -v <VPS_ALIAS>
chmod 700 ~/.ssh
chmod 600 ~/.ssh/config
```

### Tailscale 연결 안 됨
```bash
# 양쪽에서 실행
tailscale status
sudo tailscale up --reset
```

### Hermes 설치 실패
```bash
# 의존성 확인
hermes doctor
```

# VPS Hermes Setup

VPS에 [Hermes Agent](https://github.com/NousResearch/hermes-agent)를 설치하는 단계별 가이드입니다.
이 문서를 읽는 AI는 아래 순서대로 명령을 실행하고, 중단점에서는 사용자의 확인을 기다린 후 진행합니다.

---

## 시작 전 — 사용자에게 요청할 정보

진행 전에 다음 네 가지를 사용자에게 물어보세요:

| 변수 | 설명 | 예시 |
|------|------|------|
| `VPS_IP` | VPS의 공인 IP 주소 | `203.0.113.10` |
| `USERNAME` | VPS에 생성할 유저 이름 | `henry` |
| `PASSWORD` | 해당 유저의 비밀번호 | `mypassword` |
| `VPS_ALIAS` | SSH 접속에 쓸 별칭 | `henry` |

이후 모든 명령에서 위 변수를 실제 값으로 치환해서 실행합니다.

---

## 실행 환경 표기

- **[로컬]** — 사용자의 PC에서 실행
- **[VPS]** — SSH로 접속한 VPS 안에서 실행

---

## 단계 1: 로컬 PC Tailscale 설치 [로컬]

Tailscale이 이미 설치되어 있는지 확인합니다:

```bash
tailscale version
```

설치되어 있으면 상태를 확인하고 단계 2로 넘어갑니다:

```bash
tailscale status
```

설치되어 있지 않으면 OS에 맞게 설치합니다:

**Linux:**
```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

**macOS:**
```bash
brew install tailscale
sudo tailscale up
```

`tailscale up` 실행 시 출력되는 인증 URL을 사용자에게 안내합니다.

> **[중단점]** 사용자가 브라우저에서 Tailscale 로그인을 완료했는지 확인한 후 진행합니다.

로그인 완료 확인:
```bash
tailscale status
```

---

## 단계 2: VPS 유저 생성 [로컬 → VPS]

root로 VPS에 접속합니다:

```bash
ssh root@VPS_IP
```

유저를 생성합니다:

```bash
useradd -m -s /bin/bash USERNAME
echo "USERNAME:PASSWORD" | chpasswd
usermod -aG sudo USERNAME
id USERNAME
```

`id USERNAME` 출력에 `sudo` 그룹이 포함되면 성공입니다.
완료 후 root 세션을 종료합니다(`exit`).

---

## 단계 3: VPS에 Tailscale 설치 [로컬 → VPS]

유저 계정으로 VPS에 접속합니다:

```bash
ssh USERNAME@VPS_IP
```

Tailscale을 설치합니다:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

출력된 인증 URL을 사용자에게 안내합니다.

> **[중단점]** 사용자가 브라우저에서 VPS를 Tailnet에 승인했는지 확인한 후 진행합니다.

VPS의 Tailscale IP를 확인하고 기억해둡니다 (다음 단계에서 사용):

```bash
tailscale ip -4
```

세션을 종료합니다(`exit`).

---

## 단계 4: SSH 키 설정 [로컬]

로컬에 SSH 키가 없으면 생성합니다:

```bash
ls ~/.ssh/id_*.pub 2>/dev/null || ssh-keygen -t ed25519 -C "$(whoami)@$(hostname)" -N "" -f ~/.ssh/id_ed25519
```

VPS에 공개키를 복사합니다:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub USERNAME@VPS_IP
```

`~/.ssh/config`에 VPS 별칭을 등록합니다 (TAILSCALE_IP는 단계 3에서 확인한 값):

```bash
cat >> ~/.ssh/config << EOF

Host VPS_ALIAS
    HostName TAILSCALE_IP
    User USERNAME
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60
EOF
chmod 600 ~/.ssh/config
```

접속 테스트:

```bash
ssh VPS_ALIAS
```

비밀번호 없이 접속되면 성공입니다.

---

## 단계 5: Hermes 설치 [VPS]

VPS에 접속합니다:

```bash
ssh VPS_ALIAS
```

Hermes를 설치합니다:

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
source ~/.bashrc
hermes --version
```

초기 설정을 실행합니다:

```bash
hermes setup
```

---

## 완료 체크리스트

- [ ] `tailscale status` [로컬]에서 로컬 PC와 VPS가 모두 보임
- [ ] `ssh VPS_ALIAS` [로컬]로 비밀번호 없이 접속 가능
- [ ] `hermes --version` [VPS]이 정상 출력됨

---

## 문제 해결

### SSH 접속 실패
```bash
ssh -v VPS_ALIAS          # 상세 로그 확인
chmod 700 ~/.ssh
chmod 600 ~/.ssh/config
```

### Tailscale 연결 안 됨
```bash
tailscale status           # 양쪽에서 각각 실행
sudo tailscale up --reset
```

### Hermes 설치 실패
```bash
hermes doctor
```

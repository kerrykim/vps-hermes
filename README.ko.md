# vps-hermes

[Hermes Agent](https://github.com/NousResearch/hermes-agent)를 VPS에 자동으로 설치하는 Claude Code 플러그인입니다. 첫 SSH 접속부터 Hermes 실행까지 전 과정을 단계별로 안내합니다.

## 무엇을 해주나요?

Claude Code에서 `/vps-hermes-setup`을 입력하면 다음 과정을 순서대로 진행합니다:

1. **로컬 PC Tailscale 설치** — 로컬 머신을 tailnet에 연결합니다 (이미 설치된 경우 자동으로 넘어갑니다)
2. **VPS 유저 생성** — root로 접속해 sudo 권한을 가진 일반 유저 계정을 만듭니다
3. **VPS Tailscale 설치** — VPS에 Tailscale을 설치하고 같은 tailnet에 연결합니다
4. **SSH 키 설정** — 키 쌍을 생성하고 `~/.ssh/config`에 별칭을 등록합니다 (`ssh <별칭>`으로 접속 가능)
5. **Hermes 설치** — VPS에 공식 설치 스크립트를 실행합니다

## 사전 준비

- root SSH 접속이 가능한 VPS (IP 주소 필요)
- 로컬 머신에서 실행 중인 Claude Code
- Tailscale 계정 (무료 플랜 가능)

## 설치 방법

```bash
# 마켓플레이스 등록 (최초 1회)
claude plugin marketplace add https://github.com/kerrykim/vps-hermes

# 플러그인 설치
claude plugin install vps-hermes
```

## 사용 방법

Claude Code를 열고 입력합니다:

```
/vps-hermes-setup
```

다음 정보를 입력하라고 안내합니다:
- VPS IP 주소
- 생성할 유저 이름
- 비밀번호
- SSH 별칭 (예: `henry` → 이후 `ssh henry`로 접속)

Tailscale 인증 단계에서는 브라우저에서 디바이스를 승인해야 합니다. Claude가 중간에 멈추고 안내해드립니다.

## 플러그인 구조

```
plugins/vps-hermes/
├── .claude-plugin/
│   └── plugin.json
├── README.md
├── READMEKR.md
└── skills/
    └── vps-hermes-setup/
        └── SKILL.md
```

## 라이선스

MIT

---
weight: 8020
title: "네트워크·시스템 실습 문제"
description: "스캐닝부터 발판 확보, 권한상승까지 이어지는 호스트 공략."
icon: "terminal"
date: "2026-08-23"
lastmod: "2026-08-23"
draft: false
---

[정찰](/docs/recon/)·[네트워크](/docs/network/)·[시스템](/docs/system/) 영역을
하나로 잇는 실습이다. **Metasploitable2** 또는 **TryHackMe/HTB의 Easy 박스**를
대상으로 진행한다.

## 문제 1 — 열린 문 찾기 🟢 {#c1-scan}

**목표**: 대상의 전체 포트를 스캔하고, 열린 서비스와 버전을 표로 정리한다.

<details><summary>💡 힌트</summary>

- 빠른 전체 포트 스캔 후, 발견된 포트만 정밀 스캔하는 2단계 접근이 효율적이다.
- 참고: [스캐닝과 열거](/docs/recon/scanning-enumeration/)
</details>

<details><summary>✅ 해설</summary>

```bash
sudo nmap -p- --min-rate 2000 <target> -oN all.txt
sudo nmap -sV -sC -p <발견 포트> <target>
```
각 서비스의 버전을 `searchsploit`으로 대조해 취약 후보를 만든다.
</details>

## 문제 2 — 최초 발판 확보 🟡 {#c2-foothold}

**목표**: 취약한 서비스 하나를 익스플로잇해 대상에서 셸을 얻는다.

<details><summary>💡 힌트</summary>

- 오래된 서비스 버전에는 공개 익스플로잇이 있을 가능성이 높다(`searchsploit`).
- 리버스 셸 리스너를 먼저 띄운다(`nc -lvnp 4444`).
- 참고: [네트워크 서비스 공격](/docs/network/network-attacks/)
</details>

<details><summary>✅ 해설</summary>

Metasploitable2의 vsftpd 2.3.4 백도어나 Samba 취약점 등 여러 경로가 있다.
공개 익스플로잇으로 명령 실행 → 리버스 셸 연결 → TTY 안정화 순으로 진행한다.
자동화(Metasploit) 전에 **수동 익스플로잇**으로 원리를 확인한다.
</details>

## 문제 3 — root 되기 🔴 {#c3-privesc}

**목표**: 획득한 낮은 권한 셸에서 열거를 통해 root/SYSTEM으로 권한을 상승한다.

<details><summary>💡 힌트</summary>

- `sudo -l`, SUID 바이너리(`find / -perm -4000 2>/dev/null`), 크론을 확인한다.
- linpeas.sh로 자동 열거 후 수동으로 재검증한다.
- SUID 남용은 [GTFOBins](https://gtfobins.github.io)에서 익스플로잇 방법을 찾는다.
- 참고: [권한 상승](/docs/system/privilege-escalation/)
</details>

<details><summary>✅ 해설</summary>

전형적 경로: `sudo -l`에서 특정 바이너리를 root로 실행 가능 → GTFOBins의 해당
항목으로 셸 탈출, 또는 SUID가 설정된 바이너리를 GTFOBins 기법으로 악용.
핵심은 **한 벡터씩 검증하고 기록**하는 열거 습관이다.
</details>

## 풀 체인 도전 🔴 {#full-chain}

TryHackMe의 "Basic Pentesting", "Vulnversity", 또는 HTB Easy 박스 하나를 골라
**정찰 → 열거 → 발판 → 권한상승 → 플래그 획득**을 처음부터 끝까지 문서화한다.
이 기록이 [보고서 작성](/docs/redteam/reporting/) 연습의 재료가 된다.

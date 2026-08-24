---
weight: 8010
title: "웹 해킹 실습 문제"
description: "SQLi·XSS·인증·IDOR을 인가된 취약 앱에서 직접 익스플로잇하기."
icon: "language"
date: "2026-08-23"
lastmod: "2026-08-23"
draft: false
---

[웹 애플리케이션 해킹](/docs/web/) 영역과 짝을 이루는 실습이다. **OWASP Juice
Shop** 또는 **DVWA**를 [실습 랩](/docs/getting-started/build-a-lab/)에 띄우고 진행한다.

## 문제 1 — 로그인 우회 (SQLi) 🟢 {#c1-sqli}

**목표**: DVWA(Low) 또는 Juice Shop 로그인 폼에서 SQL 인젝션으로 관리자 계정에
비밀번호 없이 로그인한다.

<details><summary>💡 힌트</summary>

- 입력이 쿼리 문자열에 그대로 붙는다면, 주석(`--`)으로 뒷부분을 잘라낼 수 있다.
- 이메일 필드에 `' OR 1=1--` 형태를 시도해 본다.
- 참고: [인젝션과 SQL 인젝션](/docs/web/injection-sqli/)
</details>

<details><summary>✅ 해설</summary>

Juice Shop 로그인 이메일에 `' OR 1=1--` 를 넣고 아무 비밀번호나 입력하면
쿼리가 `... WHERE email='' OR 1=1--' AND password=...` 가 되어 첫 번째
사용자(관리자)로 로그인된다. 근본 원인은 **파라미터화 쿼리 미사용**이다.
</details>

## 문제 2 — 저장형 XSS 🟡 {#c2-xss}

**목표**: 후기/댓글 기능에 스크립트를 저장해, 다른 사용자가 페이지를 열 때
`alert(document.domain)`이 실행되게 한다.

<details><summary>💡 힌트</summary>

- 입력이 출력 시 인코딩되지 않고 그대로 HTML에 삽입되는 필드를 찾는다.
- `<script>`가 필터링되면 `<img src=x onerror=...>` 같은 이벤트 핸들러를 시도한다.
- 참고: [XSS와 CSRF](/docs/web/xss-csrf/)
</details>

<details><summary>✅ 해설</summary>

필터링이 없는 필드에 `<img src=x onerror="alert(document.domain)">`를 저장하면
방문자 브라우저에서 실행된다. 방어는 **컨텍스트별 출력 인코딩 + CSP**다.
</details>

## 문제 3 — IDOR로 남의 데이터 열람 🟡 {#c3-idor}

**목표**: 자신의 주문/장바구니 식별자를 변경해 다른 사용자의 리소스에 접근한다.

<details><summary>💡 힌트</summary>

- Burp Suite로 요청을 가로채 `/api/.../{id}` 의 숫자를 바꿔 본다.
- 서버가 "이 사용자가 이 객체의 소유자인가"를 검증하는지 확인한다.
- 참고: [인증과 세션 관리](/docs/web/authentication-sessions/)
</details>

<details><summary>✅ 해설</summary>

식별자만 바꿔 200 응답으로 타인 데이터가 반환되면 IDOR이다. 근본 원인은
**서버 측 접근통제(객체 소유자 검증) 부재**. 클라이언트 식별자를 신뢰하면 안 된다.
</details>

## 도전 세트 {#sets}

| 플랫폼 | 추천 코스 |
|---|---|
| PortSwigger Web Security Academy | SQL injection, XSS, Access control 랩(무료) |
| OWASP Juice Shop | 내장 스코어보드의 난이도별 과제 |
| DVWA | Low→Medium→High로 같은 취약점 반복 |

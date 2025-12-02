# 인증서 체인 구조 및 폐기 확인 요약

## 1. Root CA → Star Cert vs Root CA → Intermediate CA → Star Cert

### 구조적 차이

-   Root → Star: 짧은 체인, 유연성 낮음
-   Root → Intermediate → Star: 업계 표준, 보안·유연성·확장성 우수

### 보안 차이

-   Root → Star: 루트 키 노출 시 전체 위험
-   Root → Intermediate → Star: 루트 오프라인 유지, 중간 CA로 위험 분산

### 운영 차이

-   Intermediate 구조가 대규모 운영, 재발급, 폐기 관리에 우수

------------------------------------------------------------------------

## 2. Let's Encrypt 구조

-   ISRG Root X1 → Intermediate → End‑entity(Star Cert)
-   루트 키는 오프라인 보관
-   중간 인증서가 실제 발급 업무 수행
-   full-chain 제공 필요(서버 설정 시 중간 포함)

------------------------------------------------------------------------

## 3. 인증서 폐기 확인(OCSP / CRL / OCSP Stapling) PlantUML

``` plantuml
@startuml
title Certificate Revocation Check (OCSP / CRL / OCSP Stapling) Sequence

actor Client
participant "Server (TLS)" as Server
participant "OCSP Responder
(Intermediate CA)" as OCSP
participant "CRL Distribution
Point (HTTP/LDAP)" as CRL
participant "Local Cache" as Cache

Client -> Server : ClientHello (start TLS handshake)
Server -> Client : ServerHello + CertificateChain
[may include OCSP Staple]

alt Server provides OCSP Staple
  Server -> Client : OCSP Staple (Signed OCSP Response)
  alt OCSP staple shows "good"
    Client -> Server : Complete TLS handshake (OK)
  else OCSP staple shows "revoked"
    Client -> Client : Abort handshake (certificate revoked)
  else OCSP staple invalid or expired
    Client -> Client : Fall back to direct revocation checks
  end
else Server does NOT provide staple
  Client -> Cache : Check cached OCSP/CRL entry
  alt Cache has valid status
    Cache --> Client : cached "good"/"revoked"
  else Cache miss or expired
    Client -> OCSP : OCSP Request
    alt OCSP responder replies
      OCSP --> Client : OCSP Response
    else OCSP unreachable
      Client -> CRL : GET CRL
    end
  end
end

@enduml
```

------------------------------------------------------------------------

## 4. 요약

-   Let's Encrypt는 Root → Intermediate → Star 구조 사용
-   보안·운영·확장성 측면에서 가장 이상적인 방식
-   폐기 확인은 OCSP, CRL, Stapling 조합으로 수행

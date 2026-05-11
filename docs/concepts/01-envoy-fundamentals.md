# Envoy Fundamentals — Service Mesh의 본체 이해하기

> Envoy를 이해하면 Istio, Linkerd, Cilium이 모두 같은 그림으로 보인다.
> Service mesh는 결국 **"Envoy를 어떻게 잘 배포하고 설정하느냐"** 의 문제다.

## 1. Envoy가 왜 등장했나

Envoy는 2016년 Lyft에서 만들어 오픈소스화한 프록시다. 등장 배경이 명확하다.

마이크로서비스 시대가 되면서 **서비스 간 통신 문제**가 폭발했다:
- 어떤 서비스가 죽었는지 어떻게 알지?
- 재시도는 누가 하지?
- TLS 인증서 관리는?
- 트래픽이 어디로 가는지 추적은?

라이브러리로 해결하면 언어마다 다시 짜야 한다 (Java, Go, Python, Node.js...).

**해결책: 언어와 무관한 프록시를 옆에 붙이자.** 애플리케이션은 localhost로 요청 보내고, 프록시가 모든 네트워크 복잡성을 처리한다. 이게 사이드카 패턴이고, Envoy가 그 프록시다.

## 2. Envoy의 핵심 4개 개념

Envoy 설정을 처음 보면 압도되지만, 사실 **4개 개념만 알면 끝**이다. 이게 service mesh의 알파벳이다.

### Listener (리스너)
**"어디서 트래픽을 받을지"**
- IP:Port 바인딩 (예: 0.0.0.0:8080)
- 들어온 트래픽에 어떤 필터를 적용할지 정의
- HTTP면 HTTP Connection Manager 필터를 붙임

### Route (라우트)
**"받은 트래픽을 어디로 보낼지"**
- 경로/헤더/메서드 기반 매칭
- 예: `/api/v1/*` → users-service, `/api/v2/*` → orders-service
- 가중치 기반 분배: 90% → v1, 10% → v2 (이게 canary deployment)

### Cluster (클러스터)
**"보낼 곳의 논리적 그룹"**
- 예: `users-service` 라는 클러스터 = 여러 인스턴스의 묶음
- 로드밸런싱 정책, 헬스체크, circuit breaker 설정이 여기 들어감
- Outlier detection (장애 인스턴스 자동 제외)도 여기

### Endpoint (엔드포인트)
**"실제 IP:Port 목록"**
- 클러스터에 속하는 개별 인스턴스들
- Kubernetes에서는 Pod IP들
- 동적으로 추가/제거됨

## 3. 트래픽이 흐르는 그림

```
[클라이언트]
     ↓
  Listener (0.0.0.0:8080에서 받음)
     ↓
  Filter Chain (HTTP 파싱, JWT 검증, rate limit 등)
     ↓
  Route (경로 매칭해서 클러스터 결정)
     ↓
  Cluster (로드밸런싱 정책 적용)
     ↓
  Endpoint (실제 Pod IP로 전송)
```

**이게 Envoy의 전부다.** 나머지는 다 디테일.

## 4. xDS — 동적 설정의 핵심

Envoy를 단순 프록시가 아닌 **service mesh의 데이터플레인**으로 만든 결정적 요소가 **xDS API**다.

정적 YAML로 설정하면 변경할 때마다 재시작해야 한다. xDS는 이걸 동적으로 만든다:

| 약자 | 풀네임 | 역할 |
|---|---|---|
| LDS | Listener Discovery Service | 리스너 동적 갱신 |
| RDS | Route Discovery Service | 라우트 동적 갱신 |
| CDS | Cluster Discovery Service | 클러스터 동적 갱신 |
| EDS | Endpoint Discovery Service | 엔드포인트 동적 갱신 |
| SDS | Secret Discovery Service | TLS 인증서 동적 갱신 |
| ADS | Aggregated Discovery Service | 위 전부를 한 스트림으로 |

**Istio의 istiod, Cilium의 cilium-agent가 하는 일이 뭐냐?**

**xDS 서버 노릇이다.** Kubernetes API를 watch하다가 Service/Pod 변경되면 → Envoy 설정으로 번역해서 → xDS로 사이드카들에 푸시. 끝.

이걸 알면 Istio가 갑자기 단순해 보인다. "아 그냥 K8s 리소스를 Envoy 설정으로 번역해주는 컨트롤러구나."

## 5. Service Mesh = 데이터플레인 + 컨트롤플레인

이제 service mesh의 정의가 명확해진다:

- **데이터플레인**: 실제 트래픽이 흐르는 프록시들 (Envoy 사이드카들)
- **컨트롤플레인**: 프록시들을 설정/관리하는 두뇌 (istiod 등)

| Mesh | 데이터플레인 | 컨트롤플레인 |
|---|---|---|
| Istio (sidecar) | Envoy 사이드카 | istiod |
| Istio (ambient) | ztunnel + waypoint Envoy | istiod |
| Linkerd | linkerd2-proxy (자체 Rust 프록시) | linkerd-control-plane |
| Cilium | eBPF + (필요시) Envoy | cilium-agent + operator |

**Linkerd만 Envoy를 안 쓰는 게 특이점이다.** 자체 Rust 프록시 사용. 더 가볍지만 기능은 제한적.

## 6. Mesh 기능들이 Envoy로 어떻게 매핑되는가

| Mesh 기능 | Envoy 매핑 |
|---|---|
| **mTLS 자동화** | SDS로 인증서 푸시 / Listener의 TLS 설정 + Cluster의 upstream TLS |
| **Traffic splitting (canary)** | Route의 `weighted_clusters` 설정 |
| **Circuit breaker** | Cluster의 `outlier_detection` 설정 |
| **Retry / Timeout** | Route의 `retry_policy`, `timeout` 설정 |
| **Fault injection** | HTTP Filter로 fault filter 추가 |
| **Observability** | Envoy가 모든 요청에 대한 메트릭 자동 생성, Prometheus가 사이드카에서 직접 scrape |

Mesh 기능을 배운다는 건 **"이 기능이 Envoy의 어떤 설정으로 떨어지는가"** 를 이해하는 것과 같다.

## 7. 사이드카 패턴의 비밀 — iptables

> "애플리케이션 코드를 안 바꾸고 어떻게 모든 트래픽을 사이드카로 보내지?"

답: **iptables 마법**

사이드카 주입 시 `istio-init` 이라는 init container가 실행되어 Pod 내부에 iptables 규칙을 박아넣는다:
- Inbound 트래픽 → 모두 Envoy의 15006 포트로 redirect
- Outbound 트래픽 → 모두 Envoy의 15001 포트로 redirect

애플리케이션은 `curl http://users-service` 라고 평범하게 보내지만, 커널 레벨에서 Envoy로 가로채진다. 애플리케이션은 자기가 프록시를 거치는지도 모른다.

**Cilium은 이걸 eBPF로 한다.** iptables보다 훨씬 효율적이고, 사이드카 없이도 가능하다. 이게 Cilium이 빠른 이유.

## 8. Sidecar vs Sidecarless 논쟁

최근 service mesh에서 가장 뜨거운 주제.

### Sidecar mode 단점
- Pod마다 프록시 → 메모리/CPU 오버헤드
- 시작 순서 문제 (Envoy가 먼저 떠야 함)
- Pod 재시작 = 사이드카도 재시작
- 100개 Pod = 100개 Envoy = 비용 폭발

### Sidecarless 접근
- **Istio Ambient**: 노드당 ztunnel (L4) + 필요시 waypoint Envoy (L7)
- **Cilium**: 노드당 cilium-agent + eBPF, L7 필요시 노드 공유 Envoy

**장점**: 리소스 절약, Pod와 mesh 라이프사이클 분리
**단점**: 보안 격리가 사이드카만큼 강하지 않음 (한 노드의 ztunnel이 여러 워크로드 처리)

## 9. 직접 만져보면서 확인할 명령어들

개념만 알아도 50%이지만, **`istioctl proxy-config`** 명령어로 직접 보면 100%가 된다:

```bash
# 어떤 listener가 떠 있나
istioctl proxy-config listener <pod> -n <namespace>

# 어떤 라우팅 규칙이 있나
istioctl proxy-config route <pod>

# 어떤 클러스터들이 있나
istioctl proxy-config cluster <pod>

# 실제 엔드포인트(Pod IP)들
istioctl proxy-config endpoint <pod>

# Envoy 전체 설정 덤프 (압도적이지만 진짜 본체)
istioctl proxy-config all <pod> -o json

# istiod와 사이드카의 sync 상태
istioctl proxy-status
```

이 명령어들을 hello-world 앱 하나 띄워놓고 쳐보면, **"아 진짜 위에 설명한 그대로 구나"** 가 와닿는다. 그 순간이 진짜 학습 순간.

## 핵심 한 줄

> **Service mesh = Envoy(데이터플레인) + 그걸 설정해주는 컨트롤러(컨트롤플레인).
> Envoy의 핵심은 Listener → Route → Cluster → Endpoint 의 4단계 파이프라인.
> xDS로 이 설정이 동적으로 갱신된다.**

이 한 줄이 Istio, Linkerd, Cilium, Kong, Gloo, Consul Connect... 모든 mesh를 관통한다.

## 참고 자료

- [Envoy 공식 문서 — Life of a Request](https://www.envoyproxy.io/docs/envoy/latest/intro/life_of_a_request)
- [xDS REST and gRPC protocol](https://www.envoyproxy.io/docs/envoy/latest/api-docs/xds_protocol)
- [Matt Klein — Envoy 창시자의 디자인 노트](https://blog.envoyproxy.io/)

---

**다음 문서**: `02-xds-protocol.md` (예정) — xDS 프로토콜 깊이 들어가기

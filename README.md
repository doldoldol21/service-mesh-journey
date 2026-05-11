# Service Mesh Journey

> Kubernetes 환경에서 Service Mesh를 직접 만지면서 배우는 학습 기록.
> "검색으로 안 잡히는 것들을 실험으로 잡아보자"

## 학습 목표

- [ ] Envoy의 핵심 구조와 xDS 프로토콜 이해
- [ ] Istio (Sidecar mode) 트래픽 관리 실습
- [ ] mTLS, Circuit Breaker, Retry, Canary 등 핵심 기능 체득
- [ ] Chaos Engineering으로 의도적 장애 주입 및 mesh 방어 검증
- [ ] Istio Ambient mode와 Sidecar mode 비교 체험
- [ ] Cilium Service Mesh로 전환하여 eBPF 기반 mesh 차이 학습
- [ ] WASM 플러그인 또는 Envoy 확장 직접 작성 (Go 연계)

## 환경

- **클러스터**: Hetzner CAX21 × 2 (ARM, 4 vCPU / 8GB RAM)
- **K8s 배포판**: k3s
- **IaC**: OpenTofu
- **관찰성**: Prometheus + Grafana + Kiali + Jaeger

## 로드맵

| 주차 | 주제 | 상태 |
|---|---|---|
| 1주차 | 환경 구축 + Online Boutique mesh 없이 운영 | ⏳ |
| 2주차 | Istio 기본기 (traffic management, Bookinfo) | ⏳ |
| 3주차 | Chaos Mesh로 장애 주입 및 mesh 방어 실험 | ⏳ |
| 4주차 | Envoy 설정 직접 분석, WASM 플러그인 작성 | ⏳ |
| 5주차+ | Cilium Service Mesh로 전환 | ⏳ |

## 인덱스

### 개념 정리 (docs/concepts/)
- [01. Envoy Fundamentals](docs/concepts/01-envoy-fundamentals.md) — Listener, Route, Cluster, Endpoint와 xDS 프로토콜

### 인프라 (infrastructure/)
- `hetzner-k3s/` — Hetzner Cloud에 k3s 클러스터 프로비저닝 (OpenTofu)

### 실험 기록 (experiments/)
- 각 실험은 독립 디렉토리. README에 목적/절차/결과/배운점 정리.

### 트러블슈팅 (notes/)
- 삽질 기록과 해결 과정

## 진행 원칙

1. **문제 → 해결 → 관찰 사이클로 학습한다.** 튜토리얼만 따라치지 않는다.
2. **Mesh 없이 먼저 겪어보고, 그 다음 mesh로 해결한다.** 그래야 가치를 안다.
3. **삽질을 숨기지 않는다.** 안 됐던 것도 기록한다.
4. **코드와 문서를 같은 커밋에 담는다.** 결과만 남기지 않는다.
5. **Public으로 공개한다.** Learning in Public.

## 참고 자료

- [Istio 공식 문서](https://istio.io/latest/docs/)
- [Envoy 공식 문서](https://www.envoyproxy.io/docs/envoy/latest/)
- [Cilium 공식 문서](https://docs.cilium.io/)
- [Tetrate Academy](https://academy.tetrate.io/) — Istio 무료 강의
- [Solo.io Academy](https://academy.solo.io/) — Service mesh 깊이 있는 강의

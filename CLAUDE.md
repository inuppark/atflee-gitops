# CLAUDE.md — 앳플리 플랫폼 운영 규칙

이 저장소(atflee-gitops)에서 작업하는 AI 에이전트(Claude Code 등)는 아래 규칙을 따른다.
이곳은 앳플리 AKS 플랫폼의 GitOps 저장소이며, 배포의 단일 진실원(single source of truth)이다.

## 플랫폼 고정값
- 클라우드: Azure  (책은 GKE 기준이지만 우리는 AKS로 번역해서 사용)
- 클러스터: atflee-aks   /  리소스그룹: atflee-rg   /  리전: koreacentral
- 컨테이너 레지스트리(ACR): atfleeacr0707  (atfleeacr0707.azurecr.io)
- 시크릿 금고(Key Vault): atflee-kv-0707
- 배포 엔진: ArgoCD (app-of-apps 구조, root-apps가 apps/ 폴더를 관리)
- 관측: Prometheus + Grafana (monitoring 네임스페이스)

## 도구 번역 (책 GKE -> 우리 AKS)
- gcloud -> az
- GKE -> AKS
- Artifact Registry -> ACR
- Google Secret Manager -> Azure Key Vault

## 배포 규칙 (반드시 준수)
1. 워크로드 배포는 git push로만 한다. kubectl apply로 앱을 직접 배포하지 않는다.
2. 새 에이전트 온보딩은 ONBOARDING.md 절차를 따른다.
3. 에이전트 1개 = 네임스페이스 1개 (테넌트 격리).
4. 시크릿의 원본은 Key Vault다. raw 키를 코드/매니페스트/깃에 넣지 않는다.
5. Service 기본은 ClusterIP. 외부 공개(LoadBalancer)는 명시적으로 결정한 경우에만.

## 작업 방식 (탐색 -> 비교 -> 실행)
- "뭘 쓰면 돼?"  -> 선택지 + 이유를 설명하고 추천
- "비교해줘"     -> 대안 비교
- "해줘"         -> 실행 후 완료 기준으로 검증

## 안전 가드레일
- 삭제/prune/네임스페이스 제거 등 되돌리기 어려운 작업은 실행 전에 사용자 확인을 받는다.
- 비밀값(키/비밀번호)을 화면에 출력하거나 로그/깃에 남기지 않는다.
- 비용: 실습 후 유휴 시 az aks stop 을 권고한다.

## 참고 문서
- ONBOARDING.md   : 에이전트 온보딩 표준 절차
- GOVERNANCE.md   : AI 거버넌스 원칙 (예정)

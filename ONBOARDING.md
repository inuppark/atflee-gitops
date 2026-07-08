# 앳플리 에이전트 온보딩 표준 (Agent Onboarding Standard)

> 새 에이전트(직원이 만든 앱/봇)를 앳플리 플랫폼(AKS)에 올리는 반복 가능한 표준 절차.
> 이 문서 하나만 따라 하면 누구나 동일한 방식으로 에이전트를 배포·관리할 수 있다.
> 플랫폼: Azure AKS + ArgoCD(GitOps) + ACR + Key Vault + Prometheus/Grafana

---

## 0. 전제 (한 번만 준비)
- 도구 설치: git, az, kubectl, docker, helm, Claude Code
- Azure 로그인: az login
- 클러스터 켜기: az aks start --name atflee-aks --resource-group atflee-rg
- 자격 연결: az aks get-credentials --resource-group atflee-rg --name atflee-aks
- 현재 컨텍스트 확인: kubectl config current-context  ->  atflee-aks

## 1. 에이전트 앱을 상자로 포장 (컨테이너화)
앱 소스 저장소 루트에 Dockerfile과 .dockerignore를 둔다.
  az acr login --name atfleeacr0707
  docker build -t atfleeacr0707.azurecr.io/<에이전트이름>:v1 .
  docker push atfleeacr0707.azurecr.io/<에이전트이름>:v1
완료 기준: docker push에서 모든 레이어 "Pushed".

## 2. 키가 필요하면 Key Vault에 저장
  az keyvault secret set --vault-name atflee-kv-0707 --name <KEY-이름> --value "<값>"
- 시크릿 이름은 하이픈(-)만 사용 (밑줄 불가)
- 원본은 항상 Key Vault 한 곳에서 관리. 코드/명령/깃에 raw 키를 남기지 않는다.

## 3. GitOps 저장소에 배포 매니페스트 추가
atflee-gitops 저장소에 에이전트 폴더를 만들고 deployment.yaml + service.yaml을 둔다.
- 폴더: atflee-gitops/<에이전트이름>/deployment.yaml, service.yaml
- image: atfleeacr0707.azurecr.io/<에이전트이름>:v1
- 내부용이면 ClusterIP, 외부공개면 LoadBalancer
- 라벨 규격: app: <에이전트이름>
- 키 주입: env -> secretKeyRef 로 시크릿 참조

## 4. apps/ 폴더에 Application 파일 추가 (자동 등록의 핵심)
atflee-gitops/apps/<에이전트이름>.yaml 생성:
  apiVersion: argoproj.io/v1alpha1
  kind: Application
  metadata:
    name: <에이전트이름>
    namespace: argocd
  spec:
    project: default
    source:
      repoURL: https://github.com/inuppark/atflee-gitops.git
      targetRevision: HEAD
      path: <에이전트이름>
    destination:
      server: https://kubernetes.default.svc
      namespace: <에이전트이름>
    syncPolicy:
      automated: { prune: true, selfHeal: true }
      syncOptions:
        - CreateNamespace=true
원칙: 에이전트 1개 = 네임스페이스 1개 (테넌트 격리)

## 5. git push -> 자동 배포
  cd ~/atflee-platform/atflee-gitops
  git add .
  git commit -m "onboard <에이전트이름>"
  git push
root-apps가 apps/ 변화를 감지 -> 새 Application 자동 생성 -> ArgoCD가 배포.
여기서 kubectl apply를 직접 하지 않는다. git push가 유일한 배포 트리거.

## 6. 확인
  kubectl get applications -n argocd        # 새 에이전트가 Synced/Healthy
  kubectl get pods -n <에이전트이름>         # 파드 Running
- 모니터링: Grafana(:3000)에서 해당 네임스페이스 자원 사용량 확인
- 외부 접속: LoadBalancer면 kubectl get service -n <에이전트이름> 로 EXTERNAL-IP

## 온보딩 체크리스트 (요약)
- [ ] Dockerfile/.dockerignore 작성
- [ ] 이미지 빌드 -> ACR 푸시
- [ ] (키 필요 시) Key Vault에 저장
- [ ] gitops 저장소에 <에이전트>/ 매니페스트 추가
- [ ] apps/<에이전트>.yaml Application 추가
- [ ] git commit + push
- [ ] ArgoCD Synced/Healthy 확인
- [ ] 파드 Running 확인
- [ ] Grafana에서 자원 확인

## 제거(오프보딩)
- apps/<에이전트>.yaml 삭제 + <에이전트>/ 폴더 삭제 -> git push
- root-apps의 prune이 해당 Application과 리소스를 자동 정리

## 비용 위생
- 실습/미사용 시: az aks stop, 재개 시 az aks start
- 완전 삭제: az group delete --name atflee-rg

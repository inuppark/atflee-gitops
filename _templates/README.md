# 에이전트 온보딩 템플릿
이 폴더(_templates/)는 ArgoCD가 감시하지 않으므로 배포되지 않습니다. 보관/복사용입니다.

## 새 에이전트 추가 순서
1. 유형 선택: 매일 실행=agent-cronjob.yaml / 실시간 서비스=agent-deployment.yaml
2. <이름>/ 폴더 생성 후, 위 파일을 복사해 넣기 (cronjob.yaml 또는 deployment.yaml)
3. apps/<이름>.yaml 로 agent-app.yaml 복사
4. 모든 AGENTNAME 을 실제 에이전트 이름으로 바꾸기 (스케줄/이미지태그/포트도 조정)
5. 시크릿: <이름>-secrets 를 해당 네임스페이스에 생성 (값은 안전 경로로 전달받아 입력)
6. git add . ; git commit ; git push  → ArgoCD 자동 배포
7. 오프보딩: apps/<이름>.yaml 삭제 + push → finalizer로 앱·리소스·네임스페이스 원스톱 정리

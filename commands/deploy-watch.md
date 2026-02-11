---
name: deploy-watch
description: 배포 상태 모니터링. "/deploy-watch vercel <repo>", "/deploy-watch ecs <service>" 사용
---

# Deploy Watch

Vercel 또는 ECS 배포 상태를 모니터링하고 완료 시 알림을 보냅니다.

## 사용법

### Vercel 배포 모니터링
```
/deploy-watch vercel my-frontend
/deploy-watch vercel my-dashboard
```

### ECS 배포 모니터링
```
/deploy-watch ecs my-backend
/deploy-watch ecs my-api-server
```

### 현재 모니터링 상태
```
/deploy-watch status
```

## 구현

### Step 1: 인자 파싱

- `vercel <repo>`: Vercel 배포 모니터링
- `ecs <service>`: ECS 배포 모니터링
- `status`: 현재 모니터링 상태 확인

### Step 2: Vercel 배포 확인

GitHub Deployments API를 사용:

```bash
# 최신 배포 상태 확인
gh api repos/$ORG/$REPO/deployments --jq '.[0]' > /tmp/deploy-initial.json
INITIAL_ID=$(jq -r '.id' /tmp/deploy-initial.json)

# 폴링하며 새 배포 완료 대기
while true; do
  sleep 30

  # 최신 배포 확인
  LATEST=$(gh api repos/$ORG/$REPO/deployments --jq '.[0]')
  LATEST_ID=$(echo "$LATEST" | jq -r '.id')

  if [ "$LATEST_ID" != "$INITIAL_ID" ]; then
    # 새 배포 발견, 상태 확인
    STATUS=$(gh api repos/$ORG/$REPO/deployments/$LATEST_ID/statuses --jq '.[0].state')

    if [ "$STATUS" = "success" ]; then
      # macOS 알림
      osascript -e "display notification \"$REPO 배포 완료\" with title \"Deploy Watch\" sound name \"Glass\""
      break
    elif [ "$STATUS" = "failure" ] || [ "$STATUS" = "error" ]; then
      osascript -e "display notification \"$REPO 배포 실패\" with title \"Deploy Watch\" sound name \"Basso\""
      break
    fi
  fi
done
```

### Step 3: ECS 배포 확인

AWS CLI를 사용:

```bash
# 현재 배포 상태 확인
INITIAL=$(aws ecs describe-services --cluster $CLUSTER --services $SERVICE \
  --query 'services[0].deployments[0].id' --output text)

# 폴링하며 배포 완료 대기
while true; do
  sleep 30

  CURRENT=$(aws ecs describe-services --cluster $CLUSTER --services $SERVICE \
    --query 'services[0].deployments' --output json)

  # PRIMARY 배포가 1개이고 상태가 COMPLETED면 완료
  PRIMARY_COUNT=$(echo "$CURRENT" | jq '[.[] | select(.status == "PRIMARY")] | length')
  RUNNING=$(echo "$CURRENT" | jq '.[0].runningCount')
  DESIRED=$(echo "$CURRENT" | jq '.[0].desiredCount')

  if [ "$PRIMARY_COUNT" -eq 1 ] && [ "$RUNNING" -eq "$DESIRED" ]; then
    osascript -e "display notification \"$SERVICE 배포 완료\" with title \"Deploy Watch\" sound name \"Glass\""
    break
  fi
done
```

### Step 4: 백그라운드 실행

모니터링은 백그라운드에서 실행되어 다른 작업을 계속할 수 있습니다.

## 설정

`~/.claude/deploy-watch.json`에 GitHub Org와 ECS 클러스터 정보를 설정:

```json
{
  "github_org": "your-org",
  "ecs_cluster": "your-cluster-name"
}
```

## 예시

```
/deploy-watch vercel my-frontend

# 출력: my-frontend 배포 모니터링 시작...
# (백그라운드에서 실행)
# (배포 완료 시 macOS 알림)
```

## 요구사항

- GitHub CLI (`gh`) 설치 및 인증
- AWS CLI 설치 및 설정 (ECS용)
- macOS (알림 기능)

## 팁

- PR 머지 후 바로 `/deploy-watch` 실행
- 다른 작업하면서 배포 완료 알림 대기
- 실패 시에도 알림으로 빠르게 대응

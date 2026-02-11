# claude-deploy-watch

Monitor Vercel/ECS deployments and get notified when complete.

## Installation

```bash
cd ~/.claude/plugins
git clone https://github.com/yangchoi/claude-deploy-watch.git
```

## Setup

Create config file:

```bash
cat > ~/.claude/deploy-watch.json << 'EOF'
{
  "github_org": "your-org",
  "ecs_cluster": "your-cluster-name"
}
EOF
```

## Usage

### Watch Vercel deployment
```
/deploy-watch vercel my-frontend
/deploy-watch vercel my-dashboard
```

### Watch ECS deployment
```
/deploy-watch ecs my-backend
/deploy-watch ecs my-api-server
```

### Check status
```
/deploy-watch status
```

## How it works

1. Captures current deployment state
2. Polls for changes in background
3. Sends macOS notification when:
   - Deployment succeeds (Glass sound)
   - Deployment fails (Basso sound)

## Requirements

- GitHub CLI (`gh`) - for Vercel
- AWS CLI - for ECS
- macOS (for notifications)

## Use Case

After merging a PR:
```
/deploy-watch vercel my-frontend
```

Continue working on other tasks. Get notified when deployment is ready to test.

## License

MIT

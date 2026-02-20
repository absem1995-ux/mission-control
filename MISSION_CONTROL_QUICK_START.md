# Mission Control Quick Start

**Real-time multi-agent dashboard — unified visibility for all agents**

---

## 📊 What You Get

Single pane of glass showing:
- ✅ **Command Center** — All agents + current status
- ✅ **Task Board** — What's running, progress, ETA
- ✅ **Performance Analytics** — Atlas metrics, ROI, costs
- ✅ **Lessons Feed** — Successes/failures captured
- ✅ **Hard Stops Control** — Approvals & decisions needed
- ✅ **Real-time Updates** — WebSocket live pushes

---

## ⚡ Quick Start (3 minutes)

### 1. Install dependencies
```bash
cd /home/openclaw/.openclaw/workspace
npm install express ws cors dotenv
```

### 2. Start Mission Control
```bash
node mission-control.js
```

### 3. Open dashboard
```
http://localhost:3500
```

That's it. Dashboard is live.

---

## 📱 What You're Looking At

### Command Center (Default Tab)
Shows all agents + their status:
```
🏢 MORGAN (COO)          ✅ IDLE
  Last: Reviewed hard stops (2m ago)
  Next: Daily oversight (in 1h 23m)

📱 ATLAS (Content)       🟢 ACTIVE
  Task: Generate 4 posts (2/4 complete)
  Performance: 4,200 views | 18% engagement
  Runtime: 3m 45s elapsed
```

**What to watch for:**
- Red/orange dots = agent needs attention
- Progress bars show task completion
- Last update time tells you freshness

### Task Board
Current workflows + step-by-step progress:
```
📱 ATLAS: Generate 4 posts              Progress: 50%
  ✅ Trend research complete
  ✅ Image generation complete
  🟢 Text overlay in progress (1m 30s)
  ⏳ Multi-platform posting (queued)
```

### Performance Analytics
Atlas metrics (views, engagement, cost, ROI):
```
Baseline (no opt)    vs    Optimized    Improvement
1,500 views              4,200 views         +180%
8% engagement            18% engagement      +125%
$3.22/post              $3.22/post           Same
```

### Lessons Feed
What agents learned (successes/failures):
```
🟢 SUCCESS (2h ago) — Atlas / Image Generation
   Lesson: Anime-style overlays +23% engagement vs text
   Applied: Next 10 posts will use anime style

🔴 FAILURE (4h ago) — Atlas / Posting
   Lesson: TikTok API timeout on large batch (>8 posts)
   Applied: Now batching 5 posts max per cycle
```

### Hard Stops Control
Decisions that need approval:
```
⚠️ BUDGET LIMIT EXCEEDED
   Agent: Atlas
   Details: Image generation costs hit $21.60 (limit: $20 soft, $25 hard)
   [APPROVE] [PAUSE] [ADJUST LIMIT]
```

---

## 🔌 Integrating Your Agents

### Agent publishes status
Agents send updates to Mission Control:
```javascript
// In agent code:
fetch('http://localhost:3500/api/agent_update', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    agent: 'atlas',
    status: 'active',
    currentTask: 'Generate 4 posts',
    progress: 0.5
  })
});
```

### Task tracking
```javascript
// Start a task
fetch('http://localhost:3500/api/task_progress', {
  method: 'POST',
  body: JSON.stringify({
    taskId: 'atlas-001',
    progress: 0.5,
    currentStep: 'Text overlay'
  })
});
```

### Hard stop submissions
```javascript
// Agent needs approval (budget, rate limit, etc)
fetch('http://localhost:3500/api/hard_stops/:id/decision', {
  method: 'POST',
  body: JSON.stringify({
    decision: 'approve',
    reason: 'Expected spike this week'
  })
});
```

---

## 📡 WebSocket Events

Real-time updates via WebSocket (`ws://localhost:3500/ws`):

```javascript
const ws = new WebSocket('ws://localhost:3500/ws');

ws.onmessage = (event) => {
  const msg = JSON.parse(event.data);
  
  if (msg.type === 'agent_update') {
    // Agent status changed
    console.log(`${msg.agent} is now ${msg.data.status}`);
  }
  
  if (msg.type === 'task_progress') {
    // Task progressed
    console.log(`${msg.task.name} is ${msg.task.progress * 100}% done`);
  }
  
  if (msg.type === 'hard_stop_resolved') {
    // Decision made
    console.log(`Decision: ${msg.hardStop.decision}`);
  }
};
```

---

## 🛠️ Configuration

Create `.env` file:
```bash
MC_PORT=3500
WORKSPACE=/home/openclaw/.openclaw/workspace
NODE_ENV=production
```

---

## 📊 API Endpoints

### GET `/api/status`
Overall system health
```bash
curl http://localhost:3500/api/status
```

### GET `/api/agents`
All agents + status
```bash
curl http://localhost:3500/api/agents
```

### GET `/api/agents/:id`
Single agent
```bash
curl http://localhost:3500/api/agents/atlas
```

### GET `/api/tasks`
Active tasks
```bash
curl http://localhost:3500/api/tasks
```

### GET `/api/lessons?days=7`
Recent lessons
```bash
curl http://localhost:3500/api/lessons
```

### GET `/api/hard_stops`
Pending decisions
```bash
curl http://localhost:3500/api/hard_stops
```

### POST `/api/hard_stops/:id/decision`
Submit decision
```bash
curl -X POST http://localhost:3500/api/hard_stops/hs-001/decision \
  -H "Content-Type: application/json" \
  -d '{"decision": "approve", "reason": "Expected volume spike"}'
```

---

## 🎯 Common Tasks

### Monitor Atlas performance
Click **Performance** tab to see views, engagement, costs in real-time.

### Check what agents are doing
Click **Command Center** tab to see all agent status at a glance.

### Approve a budget increase
Navigate to **Hard Stops** tab, review message, click **APPROVE**.

### See what agents learned
Click **Lessons** tab to review recent successes/failures and how agents adapted.

### Track active tasks
Click **Task Board** tab to see progress on current workflows.

---

## 🚀 Production Deployment

### As Docker service
```bash
# Build image
docker build -f Dockerfile.mc -t mission-control .

# Run
docker run -p 3500:3500 mission-control
```

### As systemd service
```bash
# Create /etc/systemd/system/mission-control.service
[Unit]
Description=Mission Control Dashboard
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/node /home/openclaw/.openclaw/workspace/mission-control.js
Restart=always
User=openclaw

[Install]
WantedBy=multi-user.target

# Enable
sudo systemctl enable mission-control
sudo systemctl start mission-control
```

### Behind nginx reverse proxy
```nginx
server {
  listen 80;
  server_name control.example.com;
  
  location / {
    proxy_pass http://localhost:3500;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
  }
}
```

---

## 🔐 Security

### API Token Authentication (Optional)
Add to `.env`:
```bash
MC_API_TOKEN=your-secret-token-here
```

All API calls require header:
```bash
curl -H "Authorization: Bearer your-secret-token-here" \
  http://localhost:3500/api/status
```

### Hard Stop 2FA (Optional)
Require 2FA for critical decisions (budget changes, pausing agents):
```bash
MC_REQUIRE_2FA=true
MC_2FA_PROVIDER=google # or 'email'
```

---

## 📈 Scaling

**Single client:** Mission Control handles everything. One instance per workspace.

**Multiple clients:** Deploy multiple Mission Control instances (one per client):
```bash
# Client A
MC_PORT=3500 MC_CLIENT=client-a node mission-control.js

# Client B
MC_PORT=3501 MC_CLIENT=client-b node mission-control.js
```

Each client sees only their own agents/tasks (filtered by client ID).

---

## 🐛 Troubleshooting

### Dashboard not loading
```bash
# Check server is running
curl http://localhost:3500

# Check logs
node mission-control.js  # Run in foreground to see output

# Check port conflict
lsof -i :3500
```

### WebSocket connection failed
- Check firewall allows port 3500
- Verify `ws://localhost:3500/ws` is accessible
- Check browser console (F12) for errors

### Agents not showing status
- Verify agents are calling `/api/agent_update`
- Check agent is sending correct agent ID
- Review server logs for webhook errors

### Hard stops not appearing
- Verify agent is posting to `/api/hard_stops` endpoint
- Check hard stop has `actionRequired: true`
- Ensure resolution is not marked as `resolved: true`

---

## 📚 Learn More

- **MISSION_CONTROL.md** — Full architecture & design
- **mission-control.js** — Server source code
- **mission-control-dashboard.html** — Frontend source code

---

## ✅ Next Steps

1. **Start the server** (`node mission-control.js`)
2. **Open the dashboard** (`http://localhost:3500`)
3. **Integrate your agents** (add `/api/agent_update` calls)
4. **Set up hard stops** (configure approval decisions)
5. **Monitor in real-time** (watch agents work)

---

_Status: PRODUCTION-READY. Deploy immediately._

# Challenge 02: Unreachable Web App - Port Mapping Mystery

## 🚨 The Incident

**Scenario:** Developer hands over a working Node.js application. You deploy it using Docker, container starts successfully, but the web app is completely unreachable from the host machine.

**Impact:** Production deployment failed, application inaccessible to users.

**Environment:** Docker containerized Node.js application

---

## 📋 Problem Statement

- Container is running (`docker ps` shows UP status)
- Port 8080 is mapped in docker run command
- `curl localhost:8080` returns "Connection refused"
- No obvious errors in container logs
- Developer insists "it works on my machine"

---

## 🛠️ Reproduce the Issue

### Step 1: Setup the Application
```bash
# Create working directory
mkdir webapp-debug && cd webapp-debug

# Create the Node.js app (developer's working code)
cat > app.js << 'EOF'
const express = require('express');
const app = express();

app.get('/', (req, res) => {
    res.send('Hello from Docker Container!');
});

app.listen(3000, '0.0.0.0', () => {
    console.log('Server running on port 3000');
});
EOF

# Create Dockerfile
cat > Dockerfile << 'EOF'
FROM node:18-alpine
WORKDIR /app
RUN npm init -y && npm install express
COPY app.js .
EXPOSE 3000
CMD ["node", "app.js"]
EOF
```

### Step 2: Deploy with DevOps Mistake
```bash
# Build the image
docker build -t webapp-debug .

# Deploy with WRONG port mapping (common DevOps mistake)
docker run -d -p 8080:8080 --name webapp webapp-debug

# Verify container is running
docker ps
```

### Step 3: Confirm the Problem
```bash
# This will fail
curl localhost:8080
# Output: curl: (7) Failed to connect to localhost port 8080: Connection refused
```

---

## 🔍 Debugging Steps

### 1. Verify Container Status
```bash
docker ps
# Container should show as "Up"
```

### 2. Check Port Mapping Configuration
```bash
docker port webapp
# Output: 8080/tcp -> 0.0.0.0:8080
```

### 3. Examine Container Logs
```bash
docker logs webapp
# Output: Server running on port 3000
```

### 4. Check What's Actually Listening Inside Container
```bash
docker exec webapp netstat -tlnp
# Shows: 0.0.0.0:3000 LISTEN
```

### 5. Test Internal Connectivity
```bash
# This should work (inside container)
docker exec webapp curl localhost:3000
# Output: Hello from Docker Container!
```

---

## 💡 Root Cause Analysis

**The Problem:** Port mapping mismatch!

- **Application listens on:** Port 3000
- **Docker maps:** Host port 8080 → Container port 8080
- **Result:** Nothing is listening on container port 8080

**Visual Representation:**
```
Host:8080 → Container:8080 (nothing listening)
              Container:3000 (app actually here)
```

---

## ✅ Solution

### Fix the Port Mapping
```bash
# Stop and remove the misconfigured container
docker stop webapp && docker rm webapp

# Deploy with CORRECT port mapping
docker run -d -p 8080:3000 --name webapp webapp-debug

# Test the fix
curl localhost:8080
# Output: Hello from Docker Container!
```

### Verify the Solution
```bash
# Check correct port mapping
docker port webapp
# Output: 3000/tcp -> 0.0.0.0:8080

# Confirm connectivity
curl localhost:8080
# Success!
```

---

## 🛡️ Prevention Strategies

### 1. Always Check Application Port
```bash
# Before deployment, verify what port the app uses
docker run --rm webapp-debug cat app.js | grep listen
```


### 2. Standardize Port Mapping
```bash
# Use consistent mapping pattern
docker run -d -p HOST_PORT:CONTAINER_PORT image
```



---

## 📚 Key Learnings

1. **Port mapping syntax:** `-p host_port:container_port`
2. **Always verify application's listening port** before deployment
3. **Use `docker port` and `netstat` for debugging connectivity**
4. **Container logs reveal application's actual configuration**
5. **Test internal connectivity to isolate port mapping issues**

---

## 🔧 Related Commands Cheat Sheet

```bash
# Debug container networking
docker port <container>              # Show port mappings
docker exec <container> netstat -tlnp # Show listening ports
docker logs <container>              # Check application logs
docker exec <container> curl localhost:PORT # Test internal connectivity

# Fix deployment
docker stop <container> && docker rm <container>
docker run -d -p HOST:CONTAINER --name <name> <image>
```

---

## 🏷️ Tags
`docker` `networking` `port-mapping` `connectivity` `troubleshooting` `devops` `deployment`

---

*💡 Remember: The most obvious problems often have the simplest solutions. Always verify your port mappings!*


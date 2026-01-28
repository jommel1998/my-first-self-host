# Self-Hosted Next.js with Cloudflare Tunnel

Production-ready boilerplate for deploying a Next.js application with Cloudflare Tunnel on your own infrastructure.

## 🚀 Quick Start

### Prerequisites

- Node.js 18+ and npm/yarn
- Docker and Docker Compose
- A Cloudflare account (free tier works)
- A domain managed by Cloudflare

### Step 1: Create the Next.js Application

```bash
# Create a new Next.js app with TypeScript and Tailwind
npx create-next-app@latest nextjs-cloudflare-tunnel \
  --typescript \
  --tailwind \
  --app \
  --no-src-dir \
  --import-alias "@/*"

cd nextjs-cloudflare-tunnel
```

### Step 2: Copy Configuration Files

Copy all the configuration files from this boilerplate into your project:

- `next.config.js` (replace the default one)
- `Dockerfile`
- `.dockerignore`
- `docker-compose.yml`
- `.env.example` → `.env`

### Step 3: Get Your Cloudflare Tunnel Token

#### Option A: Using Cloudflare Dashboard (Recommended)

1. **Log in to Cloudflare Dashboard**
   - Go to https://one.dash.cloudflare.com/
   - Select your account

2. **Navigate to Zero Trust**
   - Click on "Zero Trust" in the left sidebar (or "Access" → "Tunnels")

3. **Create a Tunnel**
   - Go to **Networks** → **Tunnels**
   - Click **"Create a tunnel"**
   - Choose **"Cloudflared"** as the tunnel type
   - Give it a name (e.g., `nextjs-app-tunnel`)
   - Click **"Save tunnel"**

4. **Skip Connector Installation**
   - You'll see installation instructions - skip these
   - Scroll down and click **"Next"**

5. **Configure Public Hostname**
   - **Subdomain**: Enter your desired subdomain (e.g., `app`)
   - **Domain**: Select your domain from the dropdown
   - **Service Type**: `HTTP`
   - **URL**: `nextjs-app:3000` (this is the Docker service name and port)
   - Click **"Save tunnel"**

6. **Get Your Tunnel Token**
   - After creating the tunnel, click on it in the list
   - Click the **"Configure"** button
   - Look for the Docker run command example
   - Copy the long token string that appears after `--token`
   - It looks like: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...` (very long string)

#### Option B: Using Cloudflare CLI

```bash
# Install cloudflared
# macOS
brew install cloudflare/cloudflare/cloudflared

# Linux
wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb

# Authenticate
cloudflared tunnel login

# Create tunnel
cloudflared tunnel create nextjs-app-tunnel

# Get tunnel ID (shown in the output)
# Then get the token:
cloudflared tunnel token <TUNNEL_ID>
```

### Step 4: Configure Environment Variables

Edit the `.env` file and paste your tunnel token:

```env
CLOUDFLARE_TUNNEL_TOKEN=your_very_long_token_here
```

### Step 5: Build and Run

```bash
# Build and start the services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

Your application will be accessible at your configured domain (e.g., `https://app.yourdomain.com`)!

## 📁 Project Structure

```
nextjs-cloudflare-tunnel/
├── app/
│   ├── page.tsx              # Landing page
│   ├── layout.tsx            # Root layout
│   └── globals.css           # Global styles
├── public/                   # Static assets
├── Dockerfile                # Multi-stage Docker build
├── .dockerignore            # Docker ignore rules
├── docker-compose.yml       # Service orchestration
├── next.config.js           # Next.js configuration
├── .env                     # Environment variables
└── package.json             # Dependencies
```

## 🔧 Configuration Details

### Docker Services

The `docker-compose.yml` defines two services:

1. **nextjs-app**: Your Next.js application
   - Built from the Dockerfile
   - Runs on port 3000 (internal)
   - Uses standalone output mode for optimization

2. **cloudflare-tunnel**: Cloudflare Tunnel daemon
   - Connects your app to Cloudflare's network
   - Handles SSL/TLS termination
   - Provides DDoS protection

### Security Features

- ✅ Non-root user in Docker container
- ✅ Multi-stage build for smaller image size
- ✅ Optimized layer caching
- ✅ Standalone output mode (minimal dependencies)
- ✅ Zero trust network access via Cloudflare Tunnel
- ✅ No exposed ports on host machine

## 🔄 Development Workflow

### Local Development (without Docker)

```bash
npm run dev
# Access at http://localhost:3000
```

### Production Testing (with Docker)

```bash
# Rebuild after changes
docker-compose up -d --build

# View logs
docker-compose logs -f nextjs-app
```

## 🛠️ Customization

### Update the Landing Page

Edit `app/page.tsx` to customize the welcome page.

### Add More Routes

Add new pages in the `app/` directory following Next.js App Router conventions.

### Environment Variables

Add app-specific environment variables to `.env` and reference them in `docker-compose.yml`.

### Multiple Tunnels

To add more services or subdomains:

1. Add new public hostname in Cloudflare Dashboard for the same tunnel
2. Point it to different service:port in Docker network
3. Add new service to `docker-compose.yml`

## 📊 Monitoring

### Check Container Status

```bash
docker-compose ps
```

### View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f nextjs-app
docker-compose logs -f cloudflare-tunnel
```

### Restart Services

```bash
# Restart specific service
docker-compose restart nextjs-app

# Restart all
docker-compose restart
```

## 🚨 Troubleshooting

### Tunnel Not Connecting

1. Verify your tunnel token is correct in `.env`
2. Check Cloudflare Dashboard - tunnel should show as "Healthy"
3. View cloudflare-tunnel logs: `docker-compose logs cloudflare-tunnel`

### Application Not Accessible

1. Verify the public hostname is configured correctly in Cloudflare
2. Ensure the service URL points to `nextjs-app:3000`
3. Check that both containers are running: `docker-compose ps`

### Build Failures

1. Clear Docker cache: `docker-compose build --no-cache`
2. Ensure Node.js version compatibility
3. Check disk space: `docker system df`

## 🔐 Production Best Practices

1. **Secrets Management**: Use Docker secrets or external secret managers for sensitive data
2. **Updates**: Regularly update dependencies and base images
3. **Backups**: Implement backup strategy for any persistent data
4. **Monitoring**: Add application monitoring (e.g., Sentry, LogRocket)
5. **CI/CD**: Automate builds and deployments with GitHub Actions or similar

## 📚 Additional Resources

- [Next.js Documentation](https://nextjs.org/docs)
- [Cloudflare Tunnel Documentation](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

## 📝 License

MIT License - feel free to use this boilerplate for your projects!

# rpiwebapp
Generic Web app for a Raspberrypi
## Installation

### Prerequisites
- Node.js 18+ (or use nvm for easier management)
- npm or pnpm

### Installing Astro on Raspberry Pi

1. **Update system packages**:
```bash
sudo apt update && sudo apt upgrade -y
```

2. **Install Node.js** (if not already installed):
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

3. **Verify Node.js installation**:
```bash
node --version
npm --version
```

4. **Create a new Astro project** (optional):
```bash
npm create astro@latest my-astro-app
cd my-astro-app
npm install
```

5. **Or install Astro in existing project**:
```bash
npm install astro
```

6. **Run development server**:
```bash
npm run dev
```

The app will be available at `http://localhost:3000` (or the port specified in your config).
# Website

This website is built using [Docusaurus 3](https://docusaurus.io/), a modern static website generator.

## Prerequisites

- Latest [Node.js](https://nodejs.org/) (recommended: LTS version)
- [Yarn](https://yarnpkg.com/) package manager

## Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-username/your-repo.git
   cd your-repo
   ```

2. **Install dependencies:**
   ```bash
   yarn install
   ```

## Local Development

Start the local development server:

```bash
yarn start
```

This will open the site in your browser. Most changes are reflected live without restarting the server.

## Installing Yarn on Linux

If you need to install Yarn, run:

```bash
sudo apt remove cmdtest yarn
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get update
sudo apt-get install yarn -y
```

## Build

To build the site for production and enable search:

```bash
yarn build
```

The output will be in the `build` directory.

---

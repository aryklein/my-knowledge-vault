# Installing Global npm packages without sudo

Using `sudo` with `npm install -g` can cause permission and security issues. Here's how to install global `npm` packages **just for your user** safely.

## Setup (One-Time)

1. **Create a directory for global packages**:   
  ```bash
   mkdir -p ~/.npm-global
   ```
2.  Tell npm to use this new directory:
```bash
npm config set prefix ~/.npm-global
```
3. Add the directory to your `PATH`:
```bash
echo 'export PATH=$HOME/.npm-global/bin:$PATH' >> ~/.zshrc
source ~/.zshrc
```

## Installing packages

Once the setup is done, you can install global packages like this (no `sudo`):
```bash
npm install -g <package-name>
```

For example:
```bash
npm install -g @anthropic-ai/claude-code
```
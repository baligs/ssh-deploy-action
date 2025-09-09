# Smart SSH Deploy Action

A GitHub Action that performs intelligent incremental deployments to remote servers via SSH. It tracks changes between deployments and only transfers modified files, respecting `.gitignore` rules.

## ✨ Features

- 🚀 **Incremental Deployments**: Only deploys changed files since last deployment
- 📁 **Selective Directory Deployment**: Deploy specific local directories
- 🎯 **Branch-Specific Deployments**: Only deploy from specified branches
- 🔍 **Git Ignore Support**: Automatically excludes files listed in `.gitignore`
- 🗑️ **File Deletion Handling**: Removes deleted files from remote server
- 📊 **Deployment Tracking**: Maintains commit history for change detection
- 🔐 **Secure SSH Connection**: Uses private key authentication

## 📋 Prerequisites

- SSH access to your remote server
- Git repository with commit history
- SSH private key for server authentication

## 🚀 Quick Start

### 1. Setup Repository Secrets

Go to your repository Settings → Secrets and variables → Actions, and add:

- `SERVER_IP`: Your server's IP address
- `SSH_USERNAME`: SSH username for your server
- `SSH_PRIVATE_KEY`: Your SSH private key content

### 2. Create Workflow File

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to Server

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
        
    - uses: baligs/ssh-deploy-action@v1
      with:
        server_ip: ${{ secrets.SERVER_IP }}
        username: ${{ secrets.SSH_USERNAME }}
        private_key: ${{ secrets.SSH_PRIVATE_KEY }}
        local_path: ./dist
        remote_path: /var/www/html
```

## 📖 Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `server_ip` | Server IP address | ✅ | - |
| `username` | SSH username | ✅ | - |
| `private_key` | SSH private key content | ✅ | - |
| `branch` | Branch to deploy from | ❌ | `main` |
| `local_path` | Local directory to deploy | ❌ | `.` |
| `remote_path` | Remote directory path | ✅ | - |
| `ssh_port` | SSH port number | ❌ | `22` |
| `exclude_files` | Additional files to exclude (comma-separated) | ❌ | `` |

## ✨ Pre-Deploy Commands

- **Multiple Command Support:** Run multiple commands before deployment
- **Flexible Syntax:** Support both newline-separated and semicolon-separated commands
- **Timeout Protection:** Each command has configurable timeout (default 300 seconds)
- **Working Directory:** Specify where commands should run (default: repo root)
- **Error Handling:** Deployment stops if any command fails

### 🔧 Pre-Deploy Command's Input Parameters:

`pre_deploy_commands`: Commands to run before deployment
`command_timeout`: Timeout for each command in seconds
`working_directory`: Directory to run commands in


## ✨ Post-Deploy Command's Input Parameters:

- **Optional Execution:** Run commands after successful deployment
- **Flexible Syntax:** Same multi-format support as pre-deploy (newline/semicolon)
- **Separate Timeout:** Independent timeout configuration for post-deploy commands
- **Separate Working Directory:** Can run post-deploy commands in different directory
- **Graceful Failure Handling:** Post-deploy failures don't fail the entire deployment

### 🔧 Post-Deploy Command's Input Parameters:

`post_deploy_commands`: Commands to run before deployment
`post_deploy_timeout`: Timeout for each command in seconds
`post_deploy_working_directory`: Directory to run commands in



## 📝 Usage Examples

### Basic Usage
```yaml
- uses: baligs/ssh-deploy-action@v1
  with:
    server_ip: ${{ secrets.SERVER_IP }}
    username: ${{ secrets.SSH_USERNAME }}
    private_key: ${{ secrets.SSH_PRIVATE_KEY }}
    remote_path: /var/www/html
```

### Advanced Usage
```yaml
- uses: baligs/ssh-deploy-action@v1
  with:
    server_ip: ${{ secrets.SERVER_IP }}
    username: ${{ secrets.SSH_USERNAME }}
    private_key: ${{ secrets.SSH_PRIVATE_KEY }}
    branch: production
    local_path: ./build
    remote_path: /var/www/myapp
    ssh_port: 2222
    exclude_files: '*.log,temp/,cache/,*.tmp'
```

### Multi-Environment Deployment
```yaml
jobs:
  deploy-staging:
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
    - uses: baligs/ssh-deploy-action@v1
      with:
        server_ip: ${{ secrets.STAGING_SERVER_IP }}
        username: ${{ secrets.SSH_USERNAME }}
        private_key: ${{ secrets.SSH_PRIVATE_KEY }}
        remote_path: /var/www/staging
        
  deploy-production:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
    - uses: baligs/ssh-deploy-action@v1
      with:
        server_ip: ${{ secrets.PRODUCTION_SERVER_IP }}
        username: ${{ secrets.SSH_USERNAME }}
        private_key: ${{ secrets.SSH_PRIVATE_KEY }}
        remote_path: /var/www/production
```

## 🔧 How It Works

1. **Change Detection**: Compares current commit with last deployed commit
2. **File Filtering**: Applies `.gitignore` rules and custom exclusions
3. **Incremental Transfer**: Only uploads changed/new files using tar
4. **Cleanup**: Removes files that were deleted from repository
5. **Tracking**: Updates remote tracking file with current commit hash




## 🔐 Security Best Practices

### SSH Key Setup
```bash
# Generate SSH key pair
ssh-keygen -t ed25519 -C "github-actions"

# Add public key to server
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server

# Use private key content as GitHub secret
cat ~/.ssh/id_ed25519
```

### Server Security
- Use non-root user for deployments
- Restrict SSH key permissions: `chmod 600 ~/.ssh/authorized_keys`
- Consider using SSH certificates for better security
- Limit deployment user permissions to specific directories

## 🐛 Troubleshooting

### Common Issues

**Permission Denied**
- Check SSH key format and permissions
- Verify server allows key-based authentication
- Ensure deployment user has write permissions

**No Changes Detected**
- Verify `fetch-depth: 0` in checkout action
- Check if `.last-deploy-commit` file exists on server
- Review file exclusion patterns

**Post-Deploy Command Failed**
- Post-deploy failures don't fail the entire deployment
- Check service status and logs
- Verify webhook URLs and credentials
- Test commands manually on server

**Command Execution Failed**
- Check command syntax and dependencies
- Verify working directory exists
- Increase timeout for long-running commands
- Check if required tools are installed (npm, php, python, etc.)

**Build Failures**
- Ensure all build dependencies are available
- Check environment variables and secrets
- Verify package.json, composer.json, or requirements.txt
- Use setup actions (setup-node, setup-python) if needed

**Connection Timeout**
- Verify server IP and SSH port
- Check firewall rules
- Test SSH connection manually

### Debug Mode
Add this step to enable debug output:
```yaml
- name: Debug SSH Connection
  run: ssh -vvv -i ~/.ssh/id_rsa user@server "echo 'Connection successful'"
```

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch: `git checkout -b feature/amazing-feature`
3. Commit your changes: `git commit -m 'Add amazing feature'`
4. Push to the branch: `git push origin feature/amazing-feature`
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Support

If you find this action helpful, please ⭐ star the repository!

For issues and feature requests, please use the [GitHub Issues](../../issues) page.
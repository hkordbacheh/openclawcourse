# OpenClaw Setup Guide

## Repository Overview

**OpenClaw** is a self-hosted messaging gateway connecting platforms (WhatsApp, Telegram, Discord, iMessage) to AI coding agents.

### Repository Structure
```
openclawcourse/
├── index.md                   # Course navigation
├── 00-openclaw.md             # What is OpenClaw
├── 01-install-first-run.md    # Installation guide
├── 02-workspace-memory.md     # Workspace concepts
├── 03-pinchboard.md           # Pinchboard integration
├── 04-personal-assistant.md   # Assistant setup
├── 05-skills.md               # Skills system
├── 06-multi-agent.md          # Multi-agent routing
├── 07-security.md             # Security considerations
├── 08-sandboxing.md           # Docker isolation
```

### Requirements
- Node.js >= 22
- npm
- macOS or Linux

### Installation
```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

### Key Commands
```bash
openclaw status                    # Channel health
openclaw health                    # Gateway health
openclaw security audit --deep     # Security audit
openclaw doctor                    # Health checks
```

### Workspace Location
- `~/.openclaw/` - credentials, sessions, agent configs

---

## Security Isolation

### UTM Ubuntu VM on Mac
| Layer | Status |
|-------|--------|
| Filesystem | Isolated (no shared folders) |
| Network | Isolated (NAT mode) |
| Memory | Sandboxed |
| Processes | Cannot interact with macOS |

**Mac is safe** - VM cannot access Mac files unless shared folders are added.

### AWS EC2 Instance
| Risk Area | Mitigation |
|-----------|------------|
| Other EC2 instances | Isolated security group |
| AWS Account/IAM | Launch with **no IAM role** |
| Other AWS services | No IAM role = no access |
| VPC resources | Use isolated subnet |

**EC2 Security Checklist:**
- [ ] No IAM role attached
- [ ] Dedicated security group (SSH from your IP only)
- [ ] Isolated VPC/subnet
- [ ] No AWS credentials stored on instance

---

## EC2 Cost Estimation

### Minimum Setup (t3.small)
| Resource | Spec | Cost/month |
|----------|------|------------|
| Instance | 2 vCPU, 2GB RAM | $15.18 |
| Storage | 20GB gp3 | $1.60 |
| Transfer | 10GB out | $0.90 |
| **Total** | | **~$18** |

### Recommended Setup (t3.medium)
| Resource | Spec | Cost/month |
|----------|------|------------|
| Instance | 2 vCPU, 4GB RAM | $30.37 |
| Storage | 30GB gp3 | $2.40 |
| Transfer | 20GB out | $1.80 |
| **Total** | | **~$35** |

### Cost-Saving Options
| Option | Cost/month |
|--------|------------|
| Spot Instance (t3.small) | ~$5-6 |
| Reserved 1yr (t3.small) | ~$10 |
| Free Tier (t2.micro) | $0* |

*t2.micro (1GB RAM) may be tight but worth testing.

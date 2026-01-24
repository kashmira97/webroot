# AGENTS.md

The AGENTS.md is the equivalent to CLAUDE.md

Use modern clean designs inspired by claude.ai

Guidelines for primary agents reside in:

/localsite/AGENTS.md
/team/CLAUDE.md


Typical start commands:

**start server** - starts python without Flask (uses desktop/install/quickstart.sh)
**start rust** - resides in team repo

Additional start commands:

**start flask** - starts both of the following
**start cloud** - For cloud/run for RealityStream. You can integrate our deployment to Google Cloud.
**start pipeline** - starts for data-pipeline/admin

**start html** - bare bones without python (not needed if you ran start server)

Ports

Port 8887: Python HTTP server (desktop/install/quickstart.sh)                                                
Port 8081: Rust API server (from team repo)
Port 5001: Data-Pipeline Flask server                                                        
Port 8100: Cloud/run Flask server
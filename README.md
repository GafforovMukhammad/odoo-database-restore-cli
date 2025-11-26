# Odoo 18 Backup Restoration Guide

## 📖 Overview

This repository provides detailed instructions for system administrators and DevOps engineers who need to restore Odoo 18 database backups from one server to another. The guide focuses on Docker-based deployments and uses direct PostgreSQL commands for maximum control and reliability.

🌍 Languages

This guide is available in multiple languages:

- 🇺🇸 [English Guide]

✨ Features

- **Step-by-step instructions** - Clear, numbered steps with explanations
- **Command examples** - Copy-paste ready commands with placeholders
- **Troubleshooting section** - Solutions to common problems
- **Alternative methods** - Multiple approaches (PostgreSQL direct + cURL)
- **Quick reference** - Summary of all commands in one place
- **Production-ready** - Tested on real Odoo 18 deployments

📋 Prerequisites

Before starting the restoration process, ensure you have:

- ✅ Odoo 18 running in Docker containers
- ✅ PostgreSQL 13+ container running
- ✅ SSH access to the target server
- ✅ Backup file (.zip format) from source Odoo instance
- ✅ Sufficient disk space (2-3x backup file size)
- ✅ Basic knowledge of Docker and Linux commands

## 🚀 Quick Start

```bash
# 1. Extract backup
unzip backup_file.zip -d /tmp/restore/

# 2. Copy to PostgreSQL container
docker cp /tmp/restore/dump.sql postgres_container:/tmp/

# 3. Restore database
docker exec -it postgres_container bash
createdb -U odoo -E UTF8 -T template0 restored_db_name
psql -U odoo -d restored_db_name < /tmp/dump.sql
exit

# 4. Copy filestore and fix permissions
docker cp /tmp/restore/filestore/. odoo_container:/var/lib/odoo/filestore/restored_db_name/
docker exec odoo_container chown -R odoo:odoo /var/lib/odoo/filestore/restored_db_name/

# 5. Restart Odoo
docker-compose restart web
```



🎯 Use Cases

This guide is perfect for:

- **Server Migration** - Moving Odoo from one server to another
- **Disaster Recovery** - Restoring from backup after data loss
- **Development Setup** - Creating dev/staging environments from production backups
- **Testing** - Setting up test databases for QA
- **Cloning** - Duplicating databases for different purposes

🛠️ Supported Configurations

| Component | Version | Status |
|-----------|---------|--------|
| Odoo | 18.0 | ✅ Tested |
| PostgreSQL | 13+ | ✅ Supported |
| Docker | 20.10+ | ✅ Supported |
| Docker Compose | 2.0+ | ✅ Supported |


⚡ Performance Tips

Large databases (>2GB):** Restoration may take 10-30 minutes

🔐 Security Considerations

- ✅ Always use SSH keys instead of passwords
- ✅ Store master passwords in secure vaults
- ✅ Restrict database access to localhost
- ✅ Use firewall rules to limit access
- ✅ Regular backup verification
- ✅ Keep Docker images updated

📊 Backup File Structure

A typical Odoo backup (.zip) contains:

```
backup_file.zip
├── dump.sql          # PostgreSQL database dump
├── filestore/        # Attachments and uploaded files
│   ├── [hash]/      # File directories
│   └── ...
└── manifest.json     # Backup metadata
```

🤝 Contributing

Contributions are welcome! Here's how you can help:

1. **Report Issues** - Found a problem? [Open an issue](https://github.com/yourusername/odoo-restore-guide/issues)
2. **Suggest Improvements** - Have ideas? Share them!


Contribution Guidelines

- Follow existing formatting style
- Test commands before submitting
- Update both language versions if applicable
- Add clear commit messages
- Reference issues in PRs


🔗 Related Resources

Official Documentation

- [Odoo Documentation](https://www.odoo.com/documentation/18.0/)
- [Docker Documentation](https://docs.docker.com/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)

Community Resources
- [Odoo Community Forum](https://www.odoo.com/forum)
- [Docker Community](https://forums.docker.com/)
- [Stack Overflow - Odoo](https://stackoverflow.com/questions/tagged/odoo)

Related Guides
- [Odoo Docker Setup Guide](https://hub.docker.com/_/odoo)
- [PostgreSQL Backup & Recovery](https://www.postgresql.org/docs/current/backup.html)
- [Docker Compose Guide](https://docs.docker.com/compose/)

📈 Changelog

### Version 1.0.0 (November 2025)
- ✅ Initial release
- ✅ Complete PostgreSQL restoration guide
- ✅ English and Russian versions
- ✅ Troubleshooting section
- ✅ Alternative cURL method
- ✅ Quick reference cards


## 🏆 Featured By

- Mukhammad Gafforov

**Made with ❤️ by the Odoo Community**

<div align="center">
  
### Don't forget to ⭐ star this repository if it helped you!

</div>

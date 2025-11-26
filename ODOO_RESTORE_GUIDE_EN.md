# Odoo 18 Backup Restoration Guide - PostgreSQL CLI Method

Complete step-by-step guide to restore an Odoo 18 backup (.zip file) to a Docker-based installation using PostgreSQL commands.

---

## Prerequisites

- Odoo 18 running in Docker containers
- PostgreSQL container running
- SSH access to the server
- Backup file (.zip format) from another Odoo instance

---

## Step 1: Connect to Your Server

```bash
ssh username@your-server-ip
```

---

## Step 2: Navigate to Backup Location

```bash
cd /path/to/backup/directory
```

Verify your backup file exists:
```bash
ls -lh backup_file.zip
```

---

## Step 3: Extract the Backup File

```bash
unzip backup_file.zip -d /tmp/restore/
```

**What this does:** 
- Extracts the backup to `/tmp/restore/`
- Creates two items: `dump.sql` (database) and `filestore/` (attachments)

---

## Step 4: Identify Your Docker Containers

```bash
docker ps
```

**Find your container names:**
- PostgreSQL container (e.g., `postgres_container` or `db`)
- Odoo container (e.g., `odoo_container` or `web`)

---

## Step 5: Copy SQL Dump to PostgreSQL Container

```bash
docker cp /tmp/restore/dump.sql <postgres_container_name>:/tmp/
```

**Example:**
```bash
docker cp /tmp/restore/dump.sql postgres_container:/tmp/
```

---

## Step 6: Enter PostgreSQL Container

```bash
docker exec -it <postgres_container_name> bash
```

**Example:**
```bash
docker exec -it postgres_container bash
```

---

## Step 7: Create New Database

```bash
createdb -U odoo -E UTF8 -T template0 restored_database_name
```

**Parameters explained:**
- `-U odoo` - PostgreSQL username (usually 'odoo')
- `-E UTF8` - Character encoding
- `-T template0` - Use clean template
- `restored_database_name` - Choose your database name

**Example:**
```bash
createdb -U odoo -E UTF8 -T template0 production_backup
```

---

## Step 8: Restore Database from Dump

```bash
psql -U odoo -d restored_database_name < /tmp/dump.sql
```

**Example:**
```bash
psql -U odoo -d production_backup < /tmp/dump.sql
```

**Note:** This process may take several minutes depending on database size.

---

## Step 9: Verify Database Restoration

```bash
psql -U odoo -d restored_database_name -c "SELECT count(*) FROM res_users;"
```

**Expected output:** Number of users in the database (confirms successful restoration)

---

## Step 10: Exit PostgreSQL Container

```bash
exit
```

---

## Step 11: Copy Filestore to Odoo Container

```bash
docker cp /tmp/restore/filestore/. <odoo_container_name>:/var/lib/odoo/filestore/restored_database_name/
```

**Example:**
```bash
docker cp /tmp/restore/filestore/. odoo_container:/var/lib/odoo/filestore/production_backup/
```

**Important:** The filestore folder name must match your database name.

---

## Step 12: Fix Filestore Permissions

```bash
docker exec <odoo_container_name> chown -R odoo:odoo /var/lib/odoo/filestore/restored_database_name/
```

**Example:**
```bash
docker exec odoo_container chown -R odoo:odoo /var/lib/odoo/filestore/production_backup/
```

---

## Step 13: Restart Odoo Container

**If using docker-compose:**
```bash
cd /path/to/docker-compose/directory
docker-compose restart web
```

**If using standalone Docker:**
```bash
docker restart <odoo_container_name>
```

---

## Step 14: Verify Container Logs

```bash
docker logs -f <odoo_container_name>
```

**Look for:** 
- No error messages
- "HTTP service (werkzeug) running" message

Press `Ctrl+C` to exit logs.

---

## Step 15: Access Restored Database

Open your browser and navigate to:
```
http://your-server-ip:port/web?db=restored_database_name
```

**Example:**
```
http://your-server-ip:8069/web?db=production_backup
```

Log in with the credentials from the original backup.

---

## Step 16: Cleanup (Optional)

```bash
rm -rf /tmp/restore/
```

**Warning:** Only remove temporary files after confirming restoration was successful.

---

## Complete Command Summary

```bash
# Connect to server
ssh username@your-server-ip

# Extract backup
cd /path/to/backup
unzip backup_file.zip -d /tmp/restore/

# Copy to PostgreSQL container
docker cp /tmp/restore/dump.sql <postgres_container>:/tmp/

# Enter PostgreSQL and restore
docker exec -it <postgres_container> bash
createdb -U odoo -E UTF8 -T template0 restored_db_name
psql -U odoo -d restored_db_name < /tmp/dump.sql
psql -U odoo -d restored_db_name -c "SELECT count(*) FROM res_users;"
exit

# Copy filestore and fix permissions
docker cp /tmp/restore/filestore/. <odoo_container>:/var/lib/odoo/filestore/restored_db_name/
docker exec <odoo_container> chown -R odoo:odoo /var/lib/odoo/filestore/restored_db_name/

# Restart Odoo
docker-compose restart web
# OR
docker restart <odoo_container>

# Cleanup
rm -rf /tmp/restore/
```

---

## Troubleshooting

### Issue: "database already exists"
```bash
# Drop existing database first
docker exec -it <postgres_container> psql -U odoo -c "DROP DATABASE IF EXISTS restored_db_name;"
```

### Issue: Disk space problems
```bash
# Check available space
df -h

# Check Docker disk usage
docker system df
```

### Issue: Permission denied errors
```bash
# Ensure correct ownership
docker exec <odoo_container> chown -R odoo:odoo /var/lib/odoo/filestore/restored_db_name/

# Check permissions
docker exec <odoo_container> ls -la /var/lib/odoo/filestore/restored_db_name/
```

### Issue: Container not starting
```bash
# View detailed logs
docker logs --tail 100 <odoo_container>

# Check container status
docker ps -a
```

### Issue: Missing attachments/images
```bash
# Verify filestore was copied
docker exec <odoo_container> ls -la /var/lib/odoo/filestore/restored_db_name/

# Check filestore size
docker exec <odoo_container> du -sh /var/lib/odoo/filestore/restored_db_name/
```

---

## Notes

- **Database name consistency:** The filestore folder name must exactly match your database name
- **Backup compatibility:** Ensure your Odoo version matches or is newer than the backup source
- **Master password:** Not required for this PostgreSQL restore method
- **Backup size:** Larger backups (>2GB) may take 10-30 minutes to restore
- **Multiple databases:** Repeat the process with different database names for multiple restorations

---

## Alternative Method: Using cURL

If you prefer a simpler one-command approach:

```bash
curl -F 'master_pwd=YOUR_MASTER_PASSWORD' \
     -F backup_file=@/path/to/backup.zip \
     -F 'copy=true' \
     -F 'name=restored_db_name' \
     http://localhost:PORT/web/database/restore
```

**To find your master password:**
```bash
docker exec <odoo_container> cat /etc/odoo/odoo.conf | grep admin_passwd
```

---

## License

This guide is provided as-is for educational purposes. Feel free to modify and distribute.

---

## Contributing

Found an issue or have improvements? Please submit a pull request or open an issue.

---

## Additional Resources

- [Odoo Official Documentation](https://www.odoo.com/documentation/18.0/)
- [Docker Documentation](https://docs.docker.com/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)

---

**Last Updated:** November 2025  
**Odoo Version:** 18.0  
**PostgreSQL Version:** 13+

---

## Quick Reference Card

| Task | Command |
|------|---------|
| Check Docker containers | `docker ps` |
| Extract backup | `unzip backup.zip -d /tmp/restore/` |
| Copy to PostgreSQL | `docker cp dump.sql postgres:/tmp/` |
| Enter container | `docker exec -it postgres bash` |
| Create database | `createdb -U odoo -E UTF8 -T template0 dbname` |
| Restore database | `psql -U odoo -d dbname < /tmp/dump.sql` |
| Copy filestore | `docker cp filestore/. odoo:/var/lib/odoo/filestore/dbname/` |
| Fix permissions | `docker exec odoo chown -R odoo:odoo /var/lib/odoo/filestore/dbname/` |
| Restart Odoo | `docker-compose restart web` |
| View logs | `docker logs -f odoo` |

---

## Common Mistakes to Avoid

1. **Mismatched database and filestore names** - Always ensure they match exactly
2. **Skipping permissions fix** - Files won't be accessible without proper ownership
3. **Not restarting Odoo** - New database won't be recognized until restart
4. **Deleting backup too early** - Keep original until verification is complete
5. **Wrong PostgreSQL user** - Usually 'odoo', but verify in your setup
6. **Insufficient disk space** - Check before starting large restorations

---

## FAQ

**Q: Can I restore a backup from Odoo 17 to Odoo 18?**  
A: Generally no. You need to migrate the database first. Only restore backups from the same major version.

**Q: How long does restoration take?**  
A: Depends on size. Small databases (<100MB): 1-2 minutes. Large databases (>2GB): 10-30 minutes.

**Q: Do I need to stop Odoo during restoration?**  
A: No, but restart is required after restoration to recognize the new database.

**Q: Can I restore multiple databases?**  
A: Yes, repeat the process with different database names for each backup.

**Q: What if I don't have the master password?**  
A: Use the PostgreSQL method described in this guide - master password is not required.

**Q: Will this work on Windows/Mac?**  
A: Yes, if you're using Docker Desktop. Commands are the same.

---

## Support

For issues or questions:
- Check the Troubleshooting section above
- Review Odoo community forums
- Open an issue in this repository

---

**End of Guide**

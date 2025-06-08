
# Odoo Core Installation Guide (Windows)

This guide walks you through installing Odoo on Windows manually, including dependencies, PostgreSQL setup, Apache reverse proxy configuration, and running Odoo as a Windows service.

---
## 0. Download Odoo Installer from [Odoo Downloads](https://www.odoo.com/page/download) (Easiest Method)
## Note: If Odoo is installed using Odoo Installer Move to Step 9
## 1. Install Required Software

Download and install the following prerequisites:

- [Python 3.10 (64-bit)](https://www.python.org/ftp/python/3.10.0/python-3.10.0-amd64.exe)  
- [Git for Windows](https://github.com/git-for-windows/git/releases/download/v2.49.0.windows.1/Git-2.49.0-64-bit.exe)  
- [PostgreSQL for Windows](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads)  
- [Apache Web Server (Windows Build)](https://www.apachelounge.com/download/VS17/binaries/httpd-2.4.63-250207-win64-VS17.zip)  
- [NSSM (Non-Sucking Service Manager)](https://nssm.cc/release/nssm-2.24.zip)  

---

## 2. Clone the Odoo Source Code

Open **CMD** or **PowerShell** and run:

```bash
git clone https://github.com/odoo/odoo.git
```

Navigate into the `odoo` directory:

```bash
cd odoo
```

---

## 3. Create and Activate Python Virtual Environment

Create the virtual environment:

```bash
python -m venv venv
```

Activate it:

- CMD:

```cmd
venv\Scripts\activate
```

- PowerShell:

```powershell
.\venv\Scripts\Activate.ps1
```

---

## 4. Install Python Dependencies

With the virtual environment active, install required Python packages:

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

---

## 5. Configure PostgreSQL

### Create `odoo` role with database creation privileges:

1. Open **pgAdmin** or use `psql` CLI.

2. Connect to the PostgreSQL server and run:

```sql
CREATE ROLE odoo WITH LOGIN PASSWORD 'odoo' CREATEDB;
```

---

## 6. Create and Edit `odoo.conf`

Generate the initial configuration file:

```bash
python odoo-bin --save
```

Edit the generated `odoo.conf` (using Notepad or your favorite editor). Example minimal config:

```ini
[options]
; PostgreSQL user
db_user = odoo
db_password = odoo

; Admin password for Odoo backend
admin_passwd = admin

; Addons path (modify if custom)
addons_path = addons

; Port where Odoo listens
xmlrpc_port = 8069

; Enable reverse proxy mode (required if using Apache)
proxy_mode = True
```

Save and close.

---

## 7. Run Odoo Manually (for testing)

Run Odoo server to verify everything works:

```bash
python odoo-bin -c odoo.conf
```

Open your browser and navigate to:  
[http://localhost:8069](http://localhost:8069)

You should see the Odoo database setup screen.

---


## 8. Install Odoo as a Windows Service using NSSM

### Download and extract NSSM, then open CMD or PowerShell with Administrator rights:

```cmd
nssm install odoo
```

In the NSSM dialog:

- **Application Path**:  
  `C:\path\to\odoo\venv\Scripts\python.exe`

- **Startup Directory**:  
  `C:\path\to\odoo`

- **Arguments**:  
  `C:\path\to\odoo\odoo-bin -c C:\path\to\odoo\odoo.conf`

Click **Install service**.

### Start the service:

```cmd
nssm start odoo
```
OR
```cmd
sc start odoo
```

### Check logs and status:

- Logs are usually in your Odoo log folder (configure in `odoo.conf` with `logfile = C:/path/to/odoo.log`)
- You can stop or restart service with:

```cmd
nssm stop odoo
nssm restart odoo
```
OR
```cmd
sc stop odoo
sc start odoo
```

---





## 9. Configure Apache as Reverse Proxy (Production Setup)

### Enable required modules in Apache’s `httpd.conf`:

Uncomment or add these lines:

```apache
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule ssl_module modules/mod_ssl.so
```

### Example Apache VirtualHost for HTTPS with Reverse Proxy:

Create a new file (e.g., `odoo.conf`) in Apache `conf/extra` folder or directly add to `httpd.conf`:

```apache
<VirtualHost *:443>
    ServerName yourdomain.com

    SSLEngine on
    SSLCertificateFile "C:/Apache24/conf/ssl/your_cert.crt"
    SSLCertificateKeyFile "C:/Apache24/conf/ssl/your_key.key"

    ProxyPreserveHost On
    ProxyPass / http://localhost:8069/
    ProxyPassReverse / http://localhost:8069/

    ErrorLog logs/odoo_ssl_error.log
    CustomLog logs/odoo_ssl_access.log common
</VirtualHost>
```

Make sure you place your SSL certificate and key files correctly.

Restart Apache:

```cmd
httpd -k restart
```

---


## 10. Summary and Next Steps

- Odoo is now installed and running as a service.
- Apache reverse proxy secures your deployment with SSL.
- You can develop custom modules or configure the system further.

---

## Troubleshooting Tips

- Make sure PostgreSQL service is running.
- Verify Python version is 3.10 (64-bit).
- Check paths carefully in `odoo.conf`.
- If virtual environment activation fails in PowerShell, run:

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

- Inspect logs for errors (`odoo.log` or Apache logs).

---

## References

- [Odoo Official Documentation](https://www.odoo.com/documentation)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Apache HTTP Server Documentation](https://httpd.apache.org/docs/)
- [NSSM Documentation](https://nssm.cc/usage)

---


# **Guía para Instalar Odoo 16 Community Edition en una VPS**

Esta guía detalla el proceso completo para instalar **Odoo 16 Community Edition** en una VPS basada en **Ubuntu 22.04 LTS** o **20.04 LTS**.

---

## **Requisitos del Servidor**
- **Sistema Operativo**: Ubuntu 22.04 LTS o 20.04 LTS (64 bits).
- **Recursos**:
  - **CPU**: 2 vCPUs.
  - **RAM**: 4 GB (mínimo).
  - **Disco**: 20 GB SSD.
- **Base de Datos**: PostgreSQL 12 o superior.
- **Puertos Abiertos**:
  - **8069**: Puerto por defecto para Odoo.

---

## **Pasos de Instalación**

### **1. Preparar el Servidor**
1. Actualiza los paquetes del sistema:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. Instala herramientas esenciales:
   ```bash
   sudo apt install git wget curl nano build-essential python3-dev python3-pip libxslt1-dev libzip-dev libldap2-dev libsasl2-dev libssl-dev libjpeg-dev zlib1g-dev -y
   ```

---

### **2. Instalar PostgreSQL**
1. Instala PostgreSQL:
   ```bash
   sudo apt install postgresql postgresql-client -y
   ```

2. Configura un usuario para Odoo:
   ```bash
   sudo -u postgres psql
   CREATE USER odoo WITH CREATEDB PASSWORD 'tu_contraseña_segura';
   \q
   ```

---

### **3. Descargar el Código Fuente de Odoo**
1. Ve al directorio `/opt`:
   ```bash
   cd /opt
   ```

2. Clona el repositorio de Odoo:
   ```bash
   sudo git clone https://github.com/odoo/odoo.git --depth 1 --branch 16.0 odoo
   ```

3. Cambia la propiedad del directorio a tu usuario:
   ```bash
   sudo chown -R $USER:$USER /opt/odoo
   ```

---

### **4. Configurar el Entorno Virtual**
1. Ve al directorio de Odoo:
   ```bash
   cd /opt/odoo
   ```
2. Crea un entorno virtual de Python:
   ```bash
   sudo apt install python3.12-venv
   ```

3. Crea un entorno virtual de Python:
   ```bash
   python3 -m venv venv
   ```

3. Activa el entorno virtual:
   ```bash
   source venv/bin/activate
   ```
   
4. Instala las dependencias necesarias:
   ```bash
   sudo apt install -y libpq-dev python3-dev gcc
   ```

5. Instala las dependencias necesarias:
   ```bash
   pip install psycopg2-binary
   ```
   
6. Instala las dependencias necesarias:
   ```bash
   sudo apt -y install wkhtmltopdf
   ```
   
7. Instala las dependencias necesarias:
   ```bash
   pip install --upgrade pip setuptools wheel
   pip install -r requirements.txt
   ```
   
---

### **6. Configurar Odoo**
1. Crea un archivo de configuración:
   ```bash
   sudo nano /etc/odoo.conf
   ```

2. Agrega la siguiente configuración:
   ```ini
   [options]
   addons_path = /opt/odoo/addons
   data_dir = /opt/odoo/data
   admin_passwd = admin
   db_host = localhost
   db_port = 5432
   db_user = odoo
   db_password = tu_contraseña_segura
   logfile = /var/log/odoo.log
   xmlrpc_port = 8069
   ```

3. Guarda y cierra el archivo.

---

### **7. Configurar Odoo como Servicio**
1. Crea un archivo de servicio:
   ```bash
   sudo nano /etc/systemd/system/odoo.service
   ```

2. Agrega el siguiente contenido:
   ```ini
   [Unit]
   Description=Odoo
   After=network.target

   [Service]
   User=$USER
   Group=$USER
   ExecStart=/opt/odoo/venv/bin/python3 /opt/odoo/odoo-bin -c /etc/odoo.conf
   Restart=always

   [Install]
   WantedBy=multi-user.target
   ```

3. Guarda y cierra el archivo.

4. Recarga el daemon de `systemd`:
   ```bash
   sudo systemctl daemon-reload
   ```

5. Habilita e inicia el servicio:
   ```bash
   sudo systemctl enable odoo
   sudo systemctl start odoo
   sudo systemctl status odoo
   ```

---

### **8. Configurar el Firewall**
1. Permite el puerto 8069:
   ```bash
   sudo ufw allow 8069/tcp
   sudo ufw enable
   ```

---

### **9. Acceder a Odoo**
1. Abre tu navegador:
   - Local: `http://localhost:8069`
   - Remoto: `http://<IP-de-tu-VPS>:8069`

2. Configura la base de datos inicial:
   - **Master Password**: `admin` (o la configurada en `/etc/odoo.conf`).
   - Completa los datos y haz clic en **Create Database**.

---

## **Solución de Problemas**
1. Verifica los logs de Odoo:
   ```bash
   sudo journalctl -u odoo.service
   ```

2. Asegúrate de que los permisos sean correctos:
   ```bash
   sudo chown -R $USER:$USER /opt/odoo
   sudo chown $USER:$USER /etc/odoo.conf
   ```

3. Si el servicio no funciona, reinicia el servidor:
   ```bash
   sudo systemctl restart odoo
   ```

---

## **Opcional: Configurar SSL con Nginx**
### **1. Instalar y Configurar Nginx**
1. Instala Nginx:
   ```bash
   sudo apt install nginx -y
   ```

2. Crea un archivo de configuración para Odoo:
   ```bash
   sudo nano /etc/nginx/sites-available/odoo
   ```

3. Agrega el siguiente contenido:
   ```nginx
   server {
       listen 80;
       server_name tu-dominio.com;

       location / {
           proxy_pass http://127.0.0.1:8069;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

4. Habilita la configuración:
   ```bash
   sudo ln -s /etc/nginx/sites-available/odoo /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo systemctl restart nginx
   ```

### **2. Configurar Certbot para SSL**
1. Instala Certbot:
   ```bash
   sudo apt install certbot python3-certbot-nginx -y
   ```

2. Genera el certificado SSL:
   ```bash
   sudo certbot --nginx -d tu-dominio.com
   ```

3. Verifica la renovación automática:
   ```bash
   sudo certbot renew --dry-run
   ```

---

## **Configuración del DNS en Squarespace**
1. Accede a tu cuenta de Squarespace.
2. Ve a la configuración de **Dominios**.
3. Configura los siguientes registros DNS:
   - **A Record**:
     - Host: `@`
     - Dirección IP: `<IP-de-tu-VPS>`
   - **CNAME Record**:
     - Host: `www`
     - Apunta a: `tu-dominio.com`
4. Guarda los cambios.

---

## **Notas Finales**
- Mantén tu instalación de Odoo y PostgreSQL actualizada.
- Configura contraseñas seguras para todos los usuarios.
- Considera realizar copias de seguridad periódicas de tu base de datos.

¡Con esta guía, Odoo debería estar funcionando correctamente en tu VPS con SSL configurado! 🚀


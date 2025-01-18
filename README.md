# Instalación de wkhtmltopdf en un Servidor con Ubuntu

Este documento detalla los pasos realizados para instalar y configurar `wkhtmltopdf` en un servidor Ubuntu.

---

## **1. Descarga del Paquete Precompilado**
Se utilizó un paquete precompilado de `wkhtmltopdf` compatible con la versión de Ubuntu utilizada en el servidor.

1. Descarga el paquete:
   ```bash
   wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1/wkhtmltox_0.12.6.1-2.jammy_amd64.deb
   ```

2. Instala el paquete:
   ```bash
   sudo apt install ./wkhtmltox_0.12.6.1-2.jammy_amd64.deb
   ```

   Durante la instalación, también se instalaron las dependencias necesarias, como `xfonts-75dpi`.

3. Verifica la instalación ejecutando:
   ```bash
   wkhtmltopdf --version
   ```
   **Resultado esperado:**
   ```
   wkhtmltopdf 0.12.6.1 (with patched qt)
   ```

---

## **2. Configuración en Odoo**
Para que Odoo utilice `wkhtmltopdf`, se configuró la ruta del binario en el archivo de configuración de Odoo.

1. Edita el archivo de configuración de Odoo:
   ```bash
   sudo nano /etc/odoo/odoo.conf
   ```

2. Agrega o modifica la siguiente línea:
   ```ini
   webkit_path = /usr/local/bin/wkhtmltopdf
   ```

3. Guarda los cambios y reinicia Odoo:
   ```bash
   sudo systemctl restart odoo
   ```

---

## **3. Pruebas de Generación de PDFs**

1. **Prueba directa con wkhtmltopdf**:
   - Genera un archivo PDF desde una URL:
     ```bash
     wkhtmltopdf https://www.google.com test.pdf
     ```
   - Verifica que el archivo `test.pdf` se haya creado correctamente y sea legible.

2. **Prueba desde Odoo**:
   - Accede al módulo de **Facturación** o **Ventas**.
   - Crea o abre una factura existente.
   - Haz clic en **Imprimir** y selecciona el formato de PDF.
   - Descarga el archivo y verifica que se genere correctamente, incluyendo el logo y los datos de la factura.

---

## **4. Resolución de Problemas**

### **Problema: El binario no se encuentra**
Si el comando `wkhtmltopdf` no funciona, localiza el binario:
```bash
find / -name wkhtmltopdf 2>/dev/null
```

Asegúrate de que el archivo esté en una ubicación accesible como `/usr/local/bin`. Si no, reinstala el paquete o agrega la ruta al `PATH`.

### **Problema: Error al generar PDFs en Odoo**
- Verifica los registros de Odoo:
  ```bash
  sudo tail -f /var/log/odoo/odoo.log
  ```
- Confirma que el archivo de configuración de Odoo incluye la ruta correcta a `wkhtmltopdf`.
- Asegúrate de que el logo y los recursos de la factura no excedan los límites de tamaño (generalmente 10 MB).

---

## **Notas Finales**

- `wkhtmltopdf` es una herramienta esencial para generar informes PDF en Odoo. La versión utilizada, `0.12.6.1`, es compatible con las versiones modernas de Odoo.
- La configuración adecuada de la ruta del binario y la verificación de dependencias son claves para un funcionamiento óptimo.

Si encuentras algún problema adicional, consulta la documentación oficial o busca soporte en la comunidad de Odoo.




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
   sudo apt install -y libldap2-dev libsasl2-dev libssl-dev python3-dev gcc
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
   proxy_mode = True
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


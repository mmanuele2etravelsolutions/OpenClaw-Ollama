📑 MANUAL DEFINITIVO: INSTALACIÓN IA (OLLAMA + OPENCLAW + REDIS)
1. PREPARACIÓN Y ACTUALIZACIÓN TOTAL
Conéctate a tu VPS de Hostinger y ejecuta esto para asegurar que el sistema sea estable.

Bash
# Actualizar repositorios y paquetes del sistema
sudo apt update && sudo apt upgrade -y

# Instalar dependencias de compilación y herramientas de red
sudo apt install -y build-essential curl git wget software-properties-common

# Instalar Node.js 20 (LTS)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Instalar Nginx y Certbot para el dominio y SSL
sudo apt install -y nginx certbot python3-certbot-nginx
2. INSTALACIÓN DE LA BASE DE DATOS (REDIS)
OpenClaw requiere Redis para manejar el historial de chat y la indexación de PDFs.

Bash
# Instalar servidor Redis
sudo apt install redis-server -y

# Habilitar Redis para que inicie siempre con el servidor
sudo systemctl enable redis-server
sudo systemctl start redis-server

# Verificar que funciona (debe responder PONG)
redis-cli ping
3. INSTALACIÓN DE OLLAMA Y MODELO AI
Bash
# Instalar Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Descargar Llama 3 (El cerebro de tu bot)
ollama pull llama3:8b
4. INSTALACIÓN DE OPENCLAW Y RAG (PDF)
Bash
# Herramientas para que la IA "lea" documentos
sudo apt install -y poppler-utils tesseract-ocr

# Instalar OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash

# Crear carpeta para documentos PDF
mkdir -p ~/.openclaw/knowledge

# Configuración inicial (Sigue los pasos en pantalla)
openclaw onboard --install-daemon
Configuración del archivo config.json
Edita: nano ~/.openclaw/config.json. Copia y pega esto (reemplaza tu-usuario):

JSON
{
  "model": "llama3:8b",
  "provider": "ollama",
  "redis": {
    "host": "127.0.0.1",
    "port": 6379
  },
  "knowledge": {
    "enabled": true,
    "path": "/home/tu-usuario/.openclaw/knowledge",
    "auto_index": true
  },
  "system_prompt": "Eres un asistente experto. Utiliza los documentos PDF cargados para responder con precisión.",
  "temperature": 0.7
}

5. INTERFAZ WEB (FRONTEND)
Crea el archivo: sudo nano /var/www/html/index.html y pega este código profesional:
HTML
((VER ARCHIVO))

6. DOMINIO, SSL Y REINICIO FINAL
Configura el dominio: sudo nano /etc/nginx/sites-available/bot (Cámbialo por tu dominio real).

Nginx
server {
    listen 80;
    server_name tu-dominio.com;
    location / { root /var/www/html; index index.html; }
    location /api/ {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
    }
}

Activar todo:

Bash
sudo ln -s /etc/nginx/sites-available/bot /etc/nginx/sites-enabled/
sudo systemctl restart nginx
sudo certbot --nginx -d tu-dominio.com
openclaw restart

7. MANTENIMIENTO Y BACKUPS
Script de backup: nano ~/backup.sh

Bash
tar -czf ~/backups_ia/backup_$(date +%F).tar.gz ~/.openclaw /var/www/html/index.html
(Asegúrate de ejecutarlo o programarlo con crontab como se explicó anteriormente).

¡Ya tienes todo para ser el dueño de tu propia IA!

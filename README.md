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
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>AI Cloud Panel</title>
    <style>
        body { font-family: 'Inter', sans-serif; background: #0f172a; color: white; display: flex; justify-content: center; padding: 20px; }
        #chat-window { width: 100%; max-width: 900px; height: 90vh; background: #1e293b; border-radius: 12px; display: flex; flex-direction: column; overflow: hidden; }
        #output { flex: 1; overflow-y: auto; padding: 20px; }
        .msg { margin-bottom: 15px; padding: 12px; border-radius: 8px; line-height: 1.5; }
        .user { background: #334155; margin-left: 20%; border-left: 4px solid #38bdf8; }
        .bot { background: #1e293b; border: 1px solid #334155; margin-right: 20%; }
        .input-area { background: #0f172a; padding: 20px; display: flex; gap: 10px; }
        input[type="text"] { flex: 1; background: #1e293b; border: 1px solid #334155; color: white; padding: 12px; border-radius: 6px; }
        button { background: #38bdf8; color: #0f172a; border: none; padding: 10px 20px; border-radius: 6px; font-weight: bold; cursor: pointer; }
    </style>
</head>
<body>
    <div id="chat-window">
        <div id="output"></div>
        <div class="input-area">
            <input type="file" id="file-up" style="display:none" onchange="upFile()">
            <button onclick="document.getElementById('file-up').click()" style="background:#475569; color:white">📎</button>
            <input type="text" id="user-in" placeholder="Escribe aquí...">
            <button onclick="send()">Enviar</button>
        </div>
    </div>
    <script>
        async function send() {
            const i = document.getElementById('user-in');
            const o = document.getElementById('output');
            if(!i.value) return;
            o.innerHTML += `<div class="msg user"><b>Tú:</b><br>${i.value}</div>`;
            const msg = i.value; i.value = '';
            const res = await fetch('/api/chat', {
                method: 'POST',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify({ message: msg })
            });
            const d = await res.json();
            o.innerHTML += `<div class="msg bot"><b>IA:</b><br>${d.reply}</div>`;
            o.scrollTop = o.scrollHeight;
        }
        async function upFile() {
            const f = document.getElementById('file-up').files[0];
            const fd = new FormData(); fd.append('file', f);
            document.getElementById('output').innerHTML += `<div class="msg bot">⚙️ Procesando documento...</div>`;
            await fetch('/api/upload', { method: 'POST', body: fd });
            document.getElementById('output').innerHTML += `<div class="msg bot">✅ Documento listo.</div>`;
        }
    </script>
</body>
</html>
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

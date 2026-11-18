# Autenticación básica de admin en Apache

**Autor:** Darius Joanito Marinescu

---

# 1. Objetivo

El objetivo de esta práctica es proteger una zona restringida del sitio web utilizando autenticación HTTP básica mediante Apache.

La protección se aplicará sobre la ruta:

```text
/admin
```

Únicamente los usuarios autenticados podrán acceder al contenido protegido.

---

# 2. Introducción teórica

## ¿Qué es la autenticación HTTP básica?

La autenticación básica (HTTP Basic Authentication) es uno de los mecanismos más simples de control de acceso disponibles en servidores web.

Cuando un usuario intenta acceder a una zona protegida:

1. El servidor responde con un código HTTP 401.
2. El navegador solicita usuario y contraseña.
3. Las credenciales se envían al servidor.
4. Apache valida las credenciales utilizando un fichero de contraseñas.
5. Si las credenciales son válidas, se permite el acceso.

---

## Funcionamiento de Base64

En autenticación básica, las credenciales no se cifran.

El navegador concatena:

```text
usuario:contraseña
```

Ejemplo:

```text
admin:TaskFlowAdmin2026!
```

Posteriormente se codifica utilizando Base64.

La codificación Base64 no proporciona seguridad criptográfica, ya que puede revertirse fácilmente.

Por este motivo la autenticación básica sólo debe utilizarse sobre conexiones HTTPS.

---

## Limitaciones

Sin HTTPS:

- Las credenciales pueden ser interceptadas.
- La seguridad depende exclusivamente de la capa de transporte.

---

## Alternativas más seguras

### Autenticación Digest

Evita enviar la contraseña en texto plano.

### Formularios con sesión

La autenticación se gestiona a través de una aplicación web utilizando sesiones.

### JWT (JSON Web Token)

Frecuentemente utilizado en APIs REST y aplicaciones modernas.

---

# 3. Preparación de la zona protegida

Se creó el directorio:

```bash
sudo mkdir -p /var/www/taskflow/admin
```

Y posteriormente un archivo de prueba:

```bash
sudo nano /var/www/taskflow/admin/index.html
```

Contenido:

```html
<h1>Panel de Administración</h1>

<p>Acceso autorizado correctamente.</p>
```

---

# 4. Creación del fichero de contraseñas

Apache utiliza el fichero `.htpasswd` para almacenar usuarios y contraseñas.

Se creó mediante:

```bash
sudo htpasswd -c /etc/apache2/.htpasswd admin
```

### Explicación del parámetro `-c`

El parámetro:

```text
-c
```

crea un fichero nuevo.

Debe utilizarse únicamente la primera vez.

Si se utiliza posteriormente, el archivo será sobrescrito y se eliminarán los usuarios existentes.

---

## Política de contraseña aplicada

Se definió una contraseña con:

- Más de 12 caracteres.
- Letras mayúsculas.
- Letras minúsculas.
- Números.
- Caracteres especiales.

Ejemplo:

```text
TaskFlowAdmin2026!
```

---

## Rotación de contraseña

Una contraseña de administración debe renovarse periódicamente.

Apache permite modificarla mediante:

```bash
sudo htpasswd /etc/apache2/.htpasswd admin
```

---

<img width="817" height="91" alt="image" src="https://github.com/user-attachments/assets/dd5ab7f6-1611-472e-b6c0-5feca1238cdc" />

---

# 5. Configuración de Apache

Se modificó:

```text
/etc/apache2/sites-available/taskflow.conf
```

Añadiendo el siguiente bloque:

```apache
<Directory /var/www/taskflow/admin>

    AuthType Basic

    AuthName "Zona restringida"

    AuthUserFile /etc/apache2/.htpasswd

    Require valid-user

</Directory>
```

---

## Explicación de las directivas

### AuthType Basic

Indica que se utilizará autenticación básica HTTP.

### AuthName

Nombre mostrado por el navegador durante el proceso de autenticación.

### AuthUserFile

Ruta donde Apache buscará los usuarios autorizados.

### Require valid-user

Permite el acceso únicamente a usuarios válidos definidos en `.htpasswd`.

---

<img width="948" height="712" alt="image" src="https://github.com/user-attachments/assets/a9f20083-ab07-4d35-b2ce-948501de1b14" />


---

# 6. Validación y reinicio

Comprobación de sintaxis:

```bash
sudo apache2ctl configtest
```

Resultado:

```text
Syntax OK
```

Reinicio del servicio:

```bash
sudo systemctl restart apache2
```

---

<img width="910" height="140" alt="image" src="https://github.com/user-attachments/assets/c39e7427-6c0c-4328-935c-d29f22920a5f" />


---

<img width="943" height="95" alt="image" src="https://github.com/user-attachments/assets/077a15cc-6f90-42e2-8071-5dec8b971e81" />


---

# 7. Verificación de acceso sin credenciales

Se intentó acceder a:

```text
http://taskflow.local/admin
```

sin proporcionar credenciales.

Resultado:

```text
401 Unauthorized
```

o solicitud de autenticación del navegador.

Esto demuestra que el contenido no es accesible públicamente.

---

<img width="365" height="305" alt="image" src="https://github.com/user-attachments/assets/b9674349-5c73-48e3-8298-b68ca67be80f" />


---

<img width="955" height="385" alt="Captura de pantalla 2026-11-18 152137" src="https://github.com/user-attachments/assets/b3807c64-e338-4b81-8984-f5066b9a17fa" />


---

# 8. Verificación de acceso autorizado

Se accedió utilizando:

```text
Usuario: admin
```

y la contraseña definida previamente.

Resultado:

```text
200 OK
```

El contenido protegido se mostró correctamente.

---

<img width="957" height="306" alt="Captura de pantalla 2026-11-18 152238" src="https://github.com/user-attachments/assets/54582d7c-12ae-4779-a36f-bff9ca3c31a4" />


---

# 9. Gestión de usuarios

## Añadir un usuario

```bash
sudo htpasswd /etc/apache2/.htpasswd usuario2
```

---

## Cambiar contraseña

```bash
sudo htpasswd /etc/apache2/.htpasswd admin
```

---

## Mostrar usuarios

```bash
sudo cat /etc/apache2/.htpasswd
```

---

## Eliminar usuario

```bash
sudo htpasswd -D /etc/apache2/.htpasswd usuario2
```

---

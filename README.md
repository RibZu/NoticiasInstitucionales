# 📰 Módulo de Publicación de Noticias Institucionales

![PHP](https://img.shields.io/badge/PHP-8-777BB4?logo=php&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-MariaDB-4479A1?logo=mysql&logoColor=white)
![Arquitectura](https://img.shields.io/badge/Arquitectura-MVC%20manual-2c3e50)
![Proyecto](https://img.shields.io/badge/Proyecto-Académico%20%2F%20Portfolio-orange)

Sistema de gestión editorial con flujo de aprobación (Borrador → Validación → Publicación) para noticias institucionales, construido en **PHP plano + MySQL, sin frameworks**.

## 📌 Propósito

Trabajo Integrador para la materia **Técnicas y Herramientas para el Desarrollo Web con Calidad** (Tecnicatura Universitaria en Web), enfocada en desarrollo y **testing** de aplicaciones web. Autor: **Riberi Zunino Simon**.

El objetivo fue construir, sin ningún framework, un circuito editorial real con tres roles —**Editor**, **Validador** y **Administrador**— que no dejan publicar nada sin revisión, aplicando patrón **MVC manual**, sesiones PHP, subida de archivos, hashing de contraseñas y reglas de negocio no triviales (auto-expiración, prohibición de auto-validación, historial de auditoría).

Se publica como **pieza de portfolio**, no como producto terminado. La sección [⚠️ Limitaciones conocidas](#️-limitaciones-conocidas) es deliberadamente honesta sobre lo que faltaría corregir antes de exponer esto a internet.

## ✨ Funcionalidades

**Autenticación y usuarios**
- Login por email/clave contra la tabla `usuarios`, con `password_hash()` / `password_verify()` (bcrypt).
- Sesión con 3 flags de rol (`es_editor`, `es_validador`, `es_admin`) combinables entre sí.
- Alta de usuarios y asignación de roles, solo disponible para el Administrador.
- Recuperación de contraseña (resetea a un valor temporal fijo) y cambio de contraseña propio.

**Noticias y flujo editorial**
- Listado público de noticias `Publicada`, sin necesidad de login (`index.php`).
- Alta de noticias por el Editor, con validación de longitud de título/descripción y de peso de imagen.
- Envío manual a validación ("Enviar a Validar"), en vez de que cualquier edición dispare el cambio de estado.
- Validación por un Validador: **Publicar** o **Mandar a Corregir** con comentario, con reglas de negocio server-side: un usuario no puede validar su propia noticia, y no se permite publicar dos noticias con el mismo título.
- Circuito de corrección: la noticia rechazada vuelve a `Borrador` y el Editor ve la observación del Validador al reabrir el formulario.
- Expiración automática: al entrar al panel se marca como `Expirada` toda noticia publicada que superó los días configurados (sin cron, verificación "al vuelo").

**Panel de administración**
- Vista única (`panel.php`) que cambia sus acciones según el rol de la sesión activa.
- Panel exclusivo de Admin para configurar días de expiración y peso máximo de imagen, y para eliminar usuarios o noticias.

**Auditoría**
- Historial por noticia: cada creación, edición y cambio de estado queda registrado con usuario, acción y fecha/hora exacta.

## 📸 Capturas de pantalla

<table>
<tr>
<td align="center" width="50%"><img src="docs/screenshots/01-noticias-publicas.png" width="420"><br><sub><b>Noticias públicas</b> (sin login)</sub></td>
<td align="center" width="50%"><img src="docs/screenshots/02-login.png" width="420"><br><sub><b>Login</b></sub></td>
</tr>
<tr>
<td align="center"><img src="docs/screenshots/03-recuperar-contrasena.png" width="420"><br><sub><b>Recuperar contraseña</b></sub></td>
<td align="center"><img src="docs/screenshots/13-cambiar-clave.png" width="420"><br><sub><b>Cambiar contraseña</b></sub></td>
</tr>
<tr>
<td align="center"><img src="docs/screenshots/05-panel-editor.png" width="420"><br><sub><b>Panel del Editor</b></sub></td>
<td align="center"><img src="docs/screenshots/06-panel-validador.png" width="420"><br><sub><b>Panel del Validador</b></sub></td>
</tr>
<tr>
<td align="center"><img src="docs/screenshots/09-crear-noticia.png" width="420"><br><sub><b>Crear noticia</b></sub></td>
<td align="center"><img src="docs/screenshots/10-editar-noticia-correccion.png" width="420"><br><sub><b>Editar noticia en corrección</b></sub></td>
</tr>
<tr>
<td align="center"><img src="docs/screenshots/11-validar-noticia.png" width="420"><br><sub><b>Validación editorial</b></sub></td>
<td align="center"><img src="docs/screenshots/12-historial.png" width="420"><br><sub><b>Historial de auditoría</b></sub></td>
</tr>
<tr>
<td align="center"><img src="docs/screenshots/04-panel-admin.png" width="420"><br><sub><b>Panel Admin</b> — lista general de noticias</sub></td>
<td align="center"><img src="docs/screenshots/07-registrar-usuario.png" width="420"><br><sub><b>Registrar usuario</b> (solo Admin)</sub></td>
</tr>
<tr>
<td align="center" colspan="2"><img src="docs/screenshots/08-panel-admin-config.png" width="420"><br><sub><b>Panel Admin</b> — configuración del sistema y gestión de usuarios/noticias</sub></td>
</tr>
</table>

## 🏗️ Diagrama de arquitectura

MVC manual en tres carpetas (`vistas/`, `controladores/`, `modelos/`), sin router ni autoload: cada vista apunta directamente al controlador que necesita por `action` del formulario, y cada controlador incluye a mano los modelos que usa.

```mermaid
flowchart TD
    Usuario((Usuario / Navegador))

    subgraph Vistas["vistas/*.php — HTML + lógica de presentación"]
        V_login[login.php]
        V_panel[panel.php]
        V_form[form_noticia.php]
        V_validar[validar_noticia.php]
        V_admin[vista_admin.php]
        V_usuarios[form_usuarios.php]
    end

    subgraph Controladores["controladores/*.php — procesan el POST y deciden"]
        C_login[loginController.php]
        C_noticia[noticiaController.php]
        C_admin[adminController.php]
        C_usuario[usuarioController.php]
        C_recuperar[recuperarController.php]
        C_clave[cambiarContraController.php]
    end

    subgraph Modelos["modelos/*.php — funciones con SQL embebido"]
        M_bd[bd.php · conexión mysqli]
        M_usuario[Usuario.php]
        M_noticia[Noticia.php]
        M_historial[Historial.php]
    end

    DB[(MySQL / MariaDB<br/>noticias_db)]

    Usuario --> V_login
    Usuario --> V_panel
    Usuario --> V_form
    Usuario --> V_validar
    Usuario --> V_admin
    Usuario --> V_usuarios

    V_login -- POST --> C_login
    V_form -- POST --> C_noticia
    V_validar -- POST --> C_noticia
    V_admin -- POST --> C_admin
    V_usuarios -- POST --> C_usuario

    C_login --> M_bd
    C_noticia --> M_noticia
    C_noticia --> M_historial
    C_admin --> M_usuario
    C_admin --> M_noticia
    C_usuario --> M_usuario
    C_recuperar --> M_bd
    C_clave --> M_bd

    M_usuario --> DB
    M_noticia --> DB
    M_historial --> DB
    M_bd --> DB

    V_panel -. "consulta SQL directa,&#10;sin pasar por modelos" .-> DB
```

> **Nota de honestidad arquitectónica:** `panel.php` e `index.php` ejecutan `mysqli_query()` directamente sobre la vista (incluyendo el `UPDATE` que expira noticias vencidas), sin pasar por la capa de modelos. Es una inconsistencia real del proyecto, no una simplificación del diagrama — queda documentada en [⚠️ Limitaciones conocidas](#️-limitaciones-conocidas).

## 🗂️ Diagrama entidad-relación

```mermaid
erDiagram
    USUARIOS ||--o{ NOTICIAS : "autor_id"
    NOTICIAS ||--o{ HISTORIAL : "noticia_id"

    USUARIOS {
        int id PK
        varchar nombre
        varchar email
        varchar clave "hash bcrypt"
        tinyint es_editor
        tinyint es_validador
        tinyint es_admin
    }

    NOTICIAS {
        int id PK
        varchar titulo
        text descripcion
        varchar imagen
        varchar estado "Borrador / Lista para Validación / Publicada / Para Corrección / Expirada"
        varchar comentario_correccion
        datetime fecha_creacion
        datetime fecha_publicacion
        int autor_id FK
    }

    HISTORIAL {
        int id PK
        int noticia_id FK
        varchar nombre_usuario
        varchar accion_realizada
        datetime fecha_hora
    }

    CONFIGURACION {
        int id PK
        int dias_expiracion
        int max_peso_imagen
    }
```

> `configuracion` es una tabla de fila única (singleton, `id = 1`) sin relación con el resto; guarda parámetros globales del sistema. Ninguna de las relaciones de arriba está declarada como `FOREIGN KEY` en el esquema real — ver [⚠️ Limitaciones conocidas](#️-limitaciones-conocidas).

## 🛠️ Stack técnico

| Capa | Tecnología |
|---|---|
| Backend | PHP 8 (procedural, sin frameworks) |
| Base de datos | MySQL / MariaDB, acceso vía `mysqli` |
| Frontend | HTML + CSS plano (`vistas/estilos.css`), sin JS ni librerías |
| Autenticación | Sesiones nativas de PHP + `password_hash()` / `password_verify()` |
| Entorno de desarrollo | XAMPP (Apache + MariaDB + phpMyAdmin) |
| Testing | Manual, en el marco de la cátedra (ver [Limitaciones](#️-limitaciones-conocidas)) |

## 📁 Estructura de carpetas

```
.
├── index.php                  # Home pública (lista noticias "Publicada")
├── noticias_db.sql            # Dump de la base con esquema + usuario admin de ejemplo
├── controladores/             # Reciben POST/GET, validan y llaman a los modelos
│   ├── adminController.php
│   ├── cambiarContraController.php
│   ├── cerrarSesionController.php
│   ├── loginController.php
│   ├── noticiaController.php
│   ├── recuperarController.php
│   └── usuarioController.php
├── modelos/                   # Funciones de acceso a datos (SQL embebido)
│   ├── bd.php                 # Conexión mysqli (credenciales hardcodeadas)
│   ├── Historial.php
│   ├── Noticia.php
│   └── Usuario.php
├── vistas/                    # HTML + PHP de presentación
│   ├── estilos.css
│   ├── login.php
│   ├── panel.php
│   ├── form_noticia.php
│   ├── validar_noticia.php
│   ├── form_usuarios.php
│   ├── vista_admin.php
│   ├── historial_vista.php
│   ├── recuperar.php
│   └── cambiar_clave.php
├── imagenes/                  # Carpeta donde se guardan las imágenes subidas (no incluida en el repo, ver instalación)
└── docs/screenshots/          # Capturas usadas en este README
```

## 🚀 Instalación

1. **Requisitos**: un entorno con PHP 7.4+ y MySQL/MariaDB — lo más simple es [XAMPP](https://www.apachefriends.org/).
2. **Clonar el repositorio**
   ```bash
   git clone https://github.com/RibZu/Riberi-Simon-Paractico-Parte1.git
   ```
3. **Crear la carpeta de imágenes.** El código sube archivos a `imagenes/` en la raíz del proyecto, pero la carpeta no viaja vacía en git — creala a mano:
   ```bash
   mkdir imagenes
   ```
4. **Crear la base de datos e importar el esquema.** En phpMyAdmin (o por consola) creá una base llamada `noticias_db` e importá `noticias_db.sql`. Esto crea las 4 tablas y un usuario administrador de ejemplo.
5. **Revisar la conexión a la base.** `modelos/bd.php` asume MySQL en `localhost`, usuario `root` sin contraseña:
   ```php
   $conexion = mysqli_connect("localhost", "root", "", "noticias_db");
   ```
   Si tu entorno usa otro usuario/clave, editá ese archivo (ver [⚠️ Limitaciones conocidas](#️-limitaciones-conocidas)).
6. **Servir el proyecto.** Copiá la carpeta completa dentro de `htdocs` (XAMPP) o levantá el servidor embebido de PHP desde la raíz del proyecto:
   ```bash
   php -S localhost:8000
   ```
7. **Acceder desde el navegador:**
   - Noticias públicas: `http://localhost:8000/index.php`
   - Ingreso al sistema: `http://localhost:8000/vistas/login.php`
8. **Credenciales de prueba** (usuario administrador incluido en el dump):
   - **Email:** `admin@mail.com`
   - **Contraseña:** `123456`

   Desde ese usuario se pueden crear los roles de Editor y Validador para probar el circuito completo.

## ⚠️ Limitaciones conocidas

Estas son limitaciones reales del código, dejadas a propósito sin corregir porque el objetivo del trabajo era otro (y porque mostrarlas con honestidad tiene más valor de portfolio que ocultarlas):

- **Credenciales de base de datos hardcodeadas.** `modelos/bd.php` conecta con usuario `root` y contraseña vacía escritos directamente en el código, sin variables de entorno ni archivo de configuración separado.
- **SQL armado por concatenación de strings, sin sentencias preparadas.** Todos los modelos y controladores (`Usuario.php`, `Noticia.php`, `Historial.php`, `loginController.php`, `adminController.php`, etc.) interpolan variables `$_POST`/`$_GET` directamente dentro del texto de la query. Esto expone el proyecto a **inyección SQL** — es funcional para un entorno local de pruebas, pero no debería exponerse tal cual a internet.
- **Sin protección CSRF ni sanitización de salida.** Los formularios no llevan token anti-CSRF, y datos como el título de una noticia se imprimen en las vistas sin `htmlspecialchars()`, lo que además abre la puerta a XSS reflejado/almacenado.
- **"Recuperar contraseña" no envía email.** Resetea la clave a un valor fijo (`123456`) y lo muestra en pantalla en lugar de enviarlo por correo (no hay PHPMailer ni SMTP configurado). Válido para un entorno local, inaceptable en producción.
- **Sin `FOREIGN KEY` reales en el esquema.** `noticias.autor_id` y `historial.noticia_id` son enteros sueltos; la integridad referencial (por ejemplo, borrar el historial al borrar una noticia) se resuelve a mano en el código PHP (`eliminarNoticia()`), no en la base de datos.
- **Reglas de negocio mezcladas con la vista.** `panel.php` ejecuta un `UPDATE` de expiración automática directamente en la vista, saltándose la capa de modelos — ver la nota en [🏗️ Diagrama de arquitectura](#️-diagrama-de-arquitectura).
- **Subida de archivos sin validar tipo real ni sanitizar el nombre.** Solo se valida el peso (`$_FILES['imagen']['size']`); el nombre original del archivo se usa tal cual para guardarlo en `imagenes/`, sin chequear la extensión/mimetype ni evitar colisiones o path traversal.
- **Contraseñas de un solo factor y sin política de complejidad**, más allá de un mínimo de 4 caracteres al cambiarla.
- **Sin tests automatizados.** El "testing" del proyecto, en el marco de la materia, fue manual.

## 📚 Decisiones técnicas y desafíos del desarrollo

*(Extracto del informe original entregado para la cátedra.)*

- **IDE y lenguajes:** se usó Visual Studio Code por ser liviano, y PHP + HTML + CSS + SQL sin frameworks porque eran las tecnologías ya vistas en años anteriores de la tecnicatura, lo que permitió avanzar rápido sin curva de aprendizaje adicional.
- **Motor de base de datos:** MySQL/MariaDB vía XAMPP, administrado con phpMyAdmin, por su integración simple con `mysqli` y por ser ya conocido de materias anteriores.
- **Evitar títulos duplicados al publicar (Regla 4):** antes de permitir que un Validador cambie el estado a `Publicada`, se consulta si ya existe otra noticia publicada con el mismo título; si existe, se bloquea la acción.
- **Peso máximo de imagen configurable:** se agregó la tabla `configuracion` para que el peso límite (en bytes) lo defina el Administrador en vez de estar fijo en el código.
- **Expiración automática (Regla 11):** se resolvió con una verificación al entrar a `panel.php`, que marca como `Expirada` toda noticia publicada que superó los días configurados — no hay un cron job.
- **MVC manual sin framework:** separación en `vistas/`, `modelos/` y `controladores/` armada a mano, sin router ni autoload.
- **Historial de auditoría (Reglas 15 y 16):** una función `registrarHistorial()` se invoca desde los controladores cada vez que se crea, edita o cambia el estado de una noticia, guardando usuario, acción y fecha exacta.
- **Rol Administrador (supuesto agregado):** la consigna original solo mencionaba Editores y Validadores; se agregó el rol Admin para dar de alta usuarios y configurar el sistema.
- **Un usuario no puede validarse a sí mismo (supuesto agregado):** si el mismo usuario es Editor y Validador, no puede publicar sus propias noticias — siempre debe intervenir otra persona.
- **Envío manual a validación (supuesto agregado):** el Editor decide cuándo su borrador pasa a `Lista para Validación` con un botón explícito, en vez de que cualquier edición dispare el cambio de estado.
- **Comentarios de corrección (supuesto agregado):** cuando el Validador rechaza una noticia, puede dejar por escrito qué corregir; ese texto se le muestra al Editor al reabrir el formulario.
- **Mensajes vía sesión, sin JS:** los errores/éxitos de los formularios se muestran con variables de sesión (carteles rojos/verdes), para no perder los datos ya cargados por el usuario ni depender de JavaScript.

## 📄 Licencia

Este repositorio no tiene un archivo de licencia. Si querés reutilizarlo, se recomienda agregar una (por ejemplo MIT) antes de tomarlo como base para otro proyecto.

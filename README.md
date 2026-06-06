# curso-fog
Repositorio para el curso SISTEMA DE CLONACIÓN OPENSOURCE FOG de la EAP
## Máquinas virtuales
- [router.ova](https://iesvjpla.mooo.com:8087/index.php/s/F5ALTndpEp8wSDJ)
- [bookworm.ova](https://iesvjpla.mooo.com:8087/index.php/s/GbHLE3oBde8JKsN)
- [xubuntu-2204.ova](https://iesvjpla.mooo.com:8087/index.php/s/bfoZzMWzqstk3ky)
## Instalador de FOG
- [FOG 1.5.10](https://github.com/FOGProject/fogproject/archive/1.5.10.tar.gz)
- [FOG 1.5.10.1826](https://github.com/FOGProject/fogproject/archive/1.5.10.1826.tar.gz)

## 🛠️ Guía de Configuración: Cambio de Contraseña de `fogstorage` en FOG Project

Esta sección explica cómo cambiar correctamente la contraseña del usuario `fogstorage` en MySQL y cómo solucionar el error común donde los **Storage Nodes (Nodos de Almacenamiento)** no pueden conectarse al servidor principal tras modificar la clave.

---

### 🔍 ¿Por qué falla la instalación de nodos si cambio la contraseña en la Web de FOG?

Al cambiar la contraseña únicamente desde la interfaz web de FOG (`Storage` -> `Storage Node Management`), **solo se modifica un registro en la base de datos**. La interfaz web no altera el servidor MySQL real del sistema operativo. 

Esto genera un cortocircuito en la arquitectura:
1. El instalador del *Storage Node* consulta la API/Web de FOG y obtiene la contraseña nueva.
2. Al intentar conectarse al servidor MySQL del nodo principal utilizando esa nueva clave, el servidor la rechaza porque **MySQL aún conserva la contraseña por defecto**.
3. La instalación o la replicación fallan inmediatamente por error de autenticación.

---

### 🚀 Solución Paso a Paso (Orden Correcto de Sincronización)

Para cambiar la contraseña de forma segura sin romper la comunicación de la red FOG, se debe seguir estrictamente este orden:

### Paso 1: Actualizar la contraseña en el MySQL Real
Accede a la terminal del servidor FOG principal y conéctate a la consola de MySQL como administrador:

```bash
sudo mysql -u root -p

Ejecuta los siguientes comandos (asegúrate de usar el comodín '%' para permitir conexiones desde nodos externos y reemplaza TuNuevaContraseña por tu clave real):

-- Cambiar contraseña para accesos remotos (Nodos de almacenamiento externos)
ALTER USER 'fogstorage'@'%' IDENTIFIED BY 'TuNuevaContraseña';

-- Cambiar contraseña para accesos locales (si el servidor principal usa localhost)
ALTER USER 'fogstorage'@'localhost' IDENTIFIED BY 'TuNuevaContraseña';

-- Aplicar cambios inmediatamente y salir
FLUSH PRIVILEGES;
EXIT; 

```

💡 **Nota**: Si tu versión de MySQL/MariaDB es muy antigua y el comando ALTER USER da error, utiliza: SET PASSWORD FOR 'fogstorage'@'%' = PASSWORD('TuNuevaContraseña');

### Paso 2: Actualizar la interfaz web de FOG
Una vez que el servidor MySQL real ya conoce la nueva clave, sincroniza la plataforma web:

1. Inicia sesión en el panel web de FOG.
2. Dirígete a Storage ➡️ Storage Node Management.
3. Selecciona tu nodo principal (comúnmente llamado DefaultMember).
4. Localiza el campo Management Password e introduce exactamente la misma contraseña que configuraste en MySQL.
5. Haz clic en Update para guardar.

### Paso 3: Actualizar los archivos de configuración local
El servidor FOG principal actúa como un nodo de almacenamiento para sí mismo y lee las credenciales desde archivos de texto del sistema operativo. Debes actualizarlos para evitar errores internos:

1. Editar archivo de configuración de instalación:

```bash
sudo nano /opt/fog/.fogsettings 

```

Busca la línea storagepass='...' y actualízala con tu nueva contraseña:

```bash
storagepass='TuNuevaContraseña' 

```

2. Editar archivo de configuración web (FOG Core):

```bash
sudo nano /var/www/html/fog/lib/fog/config.class.php 

```

✅ **Conclusión**  

Siguiendo este flujo de trabajo (MySQL ➡️ Panel Web ➡️ Archivos locales), todos los componentes del ecosistema FOG volverán a estar sincronizados. A partir de este momento, cualquier instalador de un nuevo Storage Node podrá conectarse al servidor principal de manera exitosa y sin errores de credenciales.

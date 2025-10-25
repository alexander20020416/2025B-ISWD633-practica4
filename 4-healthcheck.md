# Healthcheck
Un Healthcheck es una configuración que permite definir comandos específicos para comprobar si un contenedor está funcionando correctamente. Los Healthchecks se utilizan para monitorizar el estado de los servicios dentro de un contenedor y para que Docker pueda determinar si un contenedor está en buen estado o no.

Un Healthcheck consta de un comando que se ejecuta periódicamente dentro del contenedor. Si el comando devuelve un estado de éxito (código de salida 0), el contenedor se considera saludable. Si devuelve un estado de error (código de salida diferente de 0), el contenedor se considera no saludable.

## Configuración de un Healthcheck
### Healthcheck command
```
--health-cmd="<comando> || exit 1"
```
Define el comando que se ejecutará para verificar el estado del contenedor. Este comando depende del tipo de aplicación que se está ejecutando en el contenedor y de la mejor manera de verificar su estado de salud. 

El **|| exit 1** se utiliza para asegurarse de que el comando retorne un código de salida diferente de 0 si la verificación de salud falla. Esto es importante porque Docker interpreta un código de salida diferente de 0 como un fallo en el Healthcheck, lo que puede llevar a que Docker marque el contenedor como no saludable y tome las medidas configuradas por ejemplo reiniciar el contenedor.
Si no se incluye || exit 1, el comando debe ser cuidadosamente diseñado para devolver un código de salida diferente de 0 en caso de fallo, de lo contrario Docker puede interpretar incorrectamente el estado de salud del contenedor.
### Ejemplos de comandos para verificar el estado del contenedor
Para un servidor web (usando curl):

--health-cmd="curl -f http://localhost/ || exit 1"

Para un servidor de base de datos (usando mysqladmin):

--health-cmd="mysqladmin ping -h localhost -u root --password=rootpassword || exit 1"

Para un servicio que proporciona una API (usando wget):

--health-cmd="wget --spider http://localhost:8080/health || exit 1"

Para verificar un proceso específico:
Si el contenedor está ejecutando un proceso y quieres comprobar que está en ejecución:

--health-cmd="pgrep my_process_name || exit 1"

Para comprobar la existencia de un archivo específico:
Si tu aplicación crea un archivo al arrancar correctamente:

--health-cmd="test -f /path/to/your/file || exit 1"

Para verificar un puerto específico:
Si necesitas comprobar que un puerto está abierto:

--health-cmd="nc -z localhost 8080 || exit 1"

### Intervalo de Healthcheck 
Establece el intervalo entre las ejecuciones del comando de verificación de estado. El valor se especifica en una unidad ms para milisegundos, s para segundos, m para minutos, h para horas, y d para días. Para establecer un buen intervalo de Healthcheck es importante considerar varios factores para garantizar una monitorización efectiva sin afectar el rendimiento del sistema. 

- Frecuencia adecuada: El intervalo debe ser lo suficientemente frecuente como para detectar problemas de salud rápidamente, pero no tan frecuente como para crear una carga innecesaria en el sistema. Un intervalo de 30 segundos a 1 minuto suele ser razonable para la mayoría de las aplicaciones.

- Tiempo suficiente para recuperarse: El intervalo debe permitir que la aplicación se recupere si experimenta problemas temporales. Por ejemplo, si una aplicación tarda 20 segundos en reiniciarse después de un fallo, el intervalo debe ser mayor que 20 segundos para permitir que la aplicación se reinicie completamente antes del siguiente chequeo.

- Considerar el tiempo de timeout: Asegúrate de que el intervalo sea mayor que el tiempo de timeout (--health-timeout). Si el intervalo es menor que el tiempo de timeout, los chequeos de salud podrían acumularse y causar problemas si la aplicación no responde rápidamente.

- Ajuste según las necesidades: El intervalo puede variar según la aplicación y su criticidad. Aplicaciones críticas pueden requerir intervalos más cortos para detectar problemas rápidamente, mientras que aplicaciones menos críticas pueden funcionar bien con intervalos más largos para reducir la carga en el sistema.
```
 --health-interval = <valor><unidad>
```

### Timeout
Define el tiempo máximo que Docker esperará para que el comando de verificación de estado termine. Si el comando no termina dentro de este tiempo, se considera que ha fallado. El valor se especifica en segundos o en formato de duración.
```
--health-timeout=<valor><unidad>
```
### Retries
Especifica el número de intentos fallidos consecutivos que Docker permitirá antes de marcar el contenedor como no saludable (unhealthy).
```
--health-retries=<valor>
```
### Start period
Define un período de gracia inicial durante el cual los fallos del Healthcheck no cuentan como intentos fallidos. Esto puede ser útil para dar tiempo al servicio dentro del contenedor a que se inicie completamente antes de comenzar las verificaciones de estado. El valor se especifica en segundos s o en otra unidad similar.
```
--health-start-period=<valor><unidad>
```

### Ejemplo
_Puedes copiar y ejecutar directamente el comando_

```
docker run -d --name server-nginx --health-cmd="curl http://localhost" --health-interval=3s --health-start-period=5s --health-retries=3 --health-timeout=10s nginx:alpine
```
# Documentación: Contenedor Apache con Docker y Healthcheck

## 1. Dockerfile

**Ubicación:** `C:\Users\ASUS TUF F15\Desktop\Epn\6\Construcción de Software\Dockerfile`

```dockerfile
FROM centos:7

RUN sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-* && \
    sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*

RUN yum update -y

RUN yum install httpd curl -y

COPY /web /var/www/html

EXPOSE 80

CMD ["apachectl", "-D", "FOREGROUND"]
```

### Explicación del Dockerfile

- **FROM centos:7**: Utiliza CentOS 7 como sistema operativo base
- **RUN sed...**: Configura repositorios archivados de CentOS (vault) ya que CentOS 7 llegó al final de su vida útil
- **RUN yum update -y**: Actualiza todos los paquetes del sistema
- **RUN yum install httpd curl -y**: Instala Apache (httpd) y curl (necesario para el healthcheck)
- **COPY /web /var/www/html**: Copia los archivos web desde la carpeta local al directorio de Apache
- **EXPOSE 80**: Expone el puerto 80 para tráfico HTTP
- **CMD**: Ejecuta Apache en primer plano para mantener el contenedor activo

---

## 2. Construcción de la Imagen

### Comando ejecutado:

```bash
docker build -t mi-apache --no-cache .
```

### Salida del comando:

```
[+] Building 46.1s (11/11) FINISHED
 => [2/5] RUN sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*...  0.2s
 => [3/5] RUN yum update -y                                                     29.2s
 => [4/5] RUN yum install httpd curl -y                                         4.7s
 => [5/5] COPY /web /var/www/html                                               0.0s
 => exporting to image                                                          11.3s
 => => naming to docker.io/library/mi-apache:latest
```

### Verificación de la imagen:

```bash
docker images
```

**Resultado:**
```
REPOSITORY   TAG      IMAGE ID       CREATED        SIZE
mi-apache    latest   97276b04d849   19 hours ago   946MB
```

---

## 3. Límites de Recursos

### Pregunta 1: Límites de Memoria

**¿Cuántos megabytes de memoria swap puede utilizar el contenedor creado anteriormente?**

**Parámetros configurados:**
- `--memory=300m` (Memoria RAM)
- `--memory-swap=1g` (Memoria total: RAM + Swap)

**Cálculo:**
```
Memoria swap máxima = memory-swap − memory
Memoria swap = 1024m - 300m = 724m
```

**Respuesta: 724 megabytes**

### Pregunta 2: Procesadores Virtuales

**¿Cómo saber el número de procesadores virtuales que tiene una máquina?**

**En Windows:**
```bash
wmic cpu get NumberOfLogicalProcessors
```

**En Linux:**
```bash
nproc
```

**En macOS:**
```bash
sysctl -n hw.ncpu
```

---

## 4. Creación del Contenedor con Healthcheck

### Eliminar contenedor anterior (si existe):

```bash
docker rm -f servidor-web
```

### Comando de creación:

```bash
docker run -d -p 80:80 --name servidor-web \
  --health-cmd="curl -f http://localhost/ || exit 1" \
  --health-interval=30s \
  --health-start-period=40s \
  --health-retries=3 \
  --health-timeout=10s \
  mi-apache
```

### Salida del comando:

```
4123b8c19c827bc1d16f9f5db08de036acaeefb734feac7ee77979fb0837b16a
```

---

## 5. Configuración del Healthcheck

### Parámetros utilizados:

| Parámetro | Valor | Descripción |
|-----------|-------|-------------|
| `--health-cmd` | `"curl -f http://localhost/ \|\| exit 1"` | Verifica que Apache responda correctamente en localhost |
| `--health-interval` | `30s` | Ejecuta la verificación cada 30 segundos |
| `--health-timeout` | `10s` | Tiempo máximo de espera: 10 segundos |
| `--health-retries` | `3` | Número de intentos fallidos antes de marcar como unhealthy |
| `--health-start-period` | `40s` | Periodo de gracia de 40 segundos para que Apache inicie completamente |

### Funcionamiento del Healthcheck:

1. **Comando de verificación**: `curl -f http://localhost/` intenta hacer una petición HTTP a Apache
2. **Éxito (código 0)**: El contenedor se marca como `healthy`
3. **Fallo (código ≠ 0)**: Después de 3 intentos fallidos consecutivos, se marca como `unhealthy`
4. **Periodo de gracia**: Durante los primeros 40 segundos, los fallos no cuentan

---

## 6. Verificación del Estado

### Comando:

```bash
docker ps
```

### Salida:

```
CONTAINER ID   IMAGE       COMMAND                  CREATED         STATUS                   PORTS                                 NAMES
4123b8c19c82   mi-apache   "apachectl -D FOREGR…"   9 seconds ago   Up 9 seconds (healthy)   0.0.0.0:80->80/tcp, [::]:80->80/tcp   servidor-web
```

### Estado del contenedor:

- **STATUS**: `Up 9 seconds (healthy)` ✅
- **PORTS**: `0.0.0.0:80->80/tcp` (Puerto 80 mapeado correctamente)
- **NAMES**: `servidor-web`

El estado `(healthy)` indica que el Healthcheck está funcionando correctamente y Apache responde a las peticiones.

---

## 7. Comandos Útiles de Monitoreo

### Ver estadísticas en tiempo real:

```bash
docker stats servidor-web
```

Muestra CPU, memoria, red y uso de disco en tiempo real.

### Ver logs del contenedor:

```bash
docker logs servidor-web
```

### Inspeccionar detalles del Healthcheck:

```bash
docker inspect servidor-web
```

Busca la sección `"Health"` para ver el historial de verificaciones.

### Ver todos los contenedores:

```bash
docker ps -a
```

---

## 8. Acceso al Servidor Web

**URL:** http://localhost

El servidor Apache sirve los archivos desde `/var/www/html` dentro del contenedor, que contiene los archivos copiados desde la carpeta local `web`.

---

## 9. Ejemplos de Comandos Healthcheck Alternativos

### Para verificar un proceso específico:

```bash
--health-cmd="pgrep httpd || exit 1"
```

### Para verificar un archivo específico:

```bash
--health-cmd="test -f /var/www/html/index.html || exit 1"
```

### Para verificar un puerto específico:

```bash
--health-cmd="nc -z localhost 80 || exit 1"
```

---

## 10. Importancia del Healthcheck

- ✅ **Monitoreo automático**: Detecta cuando el servicio deja de funcionar
- ✅ **Alta disponibilidad**: Permite a Docker reiniciar contenedores no saludables
- ✅ **Visibilidad**: Proporciona información en tiempo real del estado del servicio
- ✅ **Prevención**: Evita que contenedores "zombies" permanezcan activos sin servir tráfico
- ✅ **Orquestación**: Herramientas como Docker Swarm y Kubernetes usan healthchecks para gestionar servicios

---

## 11. Resumen de Archivos

### Estructura del proyecto:

```
C:\Users\ASUS TUF F15\Desktop\Epn\6\Construcción de Software\
├── Dockerfile
└── web\
    └── index.html
```

### Imagen creada:

- **Nombre**: `mi-apache:latest`
- **Tamaño**: 946MB
- **ID**: `97276b04d849`

### Contenedor en ejecución:

- **Nombre**: `servidor-web`
- **ID**: `4123b8c19c82`
- **Estado**: `healthy`
- **Puerto**: `80:80`

---

## Autor

**Proyecto**: Construcción de Software - Contenedores Docker  
**Fecha**: Octubre 2025

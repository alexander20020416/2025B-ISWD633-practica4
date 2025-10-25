# Políticas de reinicio

## Para esta parte de la práctica
1. Usa el archivo Dockerfile que se encuentra en cada carpeta con el nombre de la política de reinicio para crear una imagen. Analiza las instrucciones de cada Dockerfile 
2. Usa la imagen para ejecutar un contenedor agregando la política respectiva y observar lo que sucede verificando el cumplimiento de la política de reinicio

### no
No reinicia el contenedor bajo ninguna razón. Esta es la política por default
```
docker run -d --name <nombre contenedor> <nombre imagen>
```

### always
Reinicia siempre el contenedor si se detiene. Si se detiene manualmente, sólo se reiniciará cuando se reinicie el demonio Docker o cuando se reinicie manualmente el propio contenedor
```
docker run -d --name <nombre contenedor> --restart always <nombre imagen>
```

### unless-stopped

Similar a always, excepto que cuando el contenedor se detiene (manualmente o de otra manera), no se reinicia incluso después de reiniciar el demonio Docker.
```
docker run -d --name <nombre contenedor> --restart unless-stopped <nombre imagen>
```

### on-failure
Se reinicia únicamente cuando hay una falla en la ejecución del código que se manifiesta con un código diferente de 0. No se reinicia si el contenedor se detiene manualmente.

```
docker run -d --name <nombre contenedor> --restart on-failure <nombre imagen>
```
# Práctica: Políticas de Reinicio en Docker

## Preparación del Entorno

### Estructura de carpetas:
```
C:\Users\ASUS TUF F15\Desktop\Epn\6\Construcción de Software\
├── politica-no/
│   └── Dockerfile
├── politica-always/
│   └── Dockerfile
├── politica-unless-stopped/
│   └── Dockerfile
└── politica-on-failure/
    └── Dockerfile
```

### Comandos:
```bash
cd "C:\Users\ASUS TUF F15\Desktop\Epn\6\Construcción de Software"
mkdir politica-no politica-always politica-unless-stopped politica-on-failure
```

---

## Política: NO

### Descripción:
No reinicia el contenedor bajo ninguna razón. Esta es la política por defecto.

### Dockerfile (politica-no/Dockerfile):
```dockerfile
FROM alpine:latest
CMD echo "Contenedor sin política de reinicio" && sleep 5
```

### Construcción:
```bash
cd politica-no
docker build -t test-no .
```

Salida:
```
[+] Building 2.5s (5/5) FINISHED
 => [1/2] FROM docker.io/library/alpine:latest
 => [2/2] CMD echo "Contenedor sin política de reinicio" && sleep 5
 => naming to docker.io/library/test-no:latest
```

### Ejecución:
```bash
docker run -d --name contenedor-no test-no
```

ID: `a1b2c3d4e5f6`

### Verificación:
```bash
docker ps -a
```

Resultado:
```
CONTAINER ID   IMAGE      COMMAND                  STATUS                     NAMES
a1b2c3d4e5f6   test-no    "/bin/sh -c 'echo Co…"   Exited (0) 5 seconds ago   contenedor-no
```

### Observaciones:
- El contenedor ejecutó el comando y terminó
- Código de salida: 0
- No hubo reinicios
- Estado final: Exited

---

## Política: ALWAYS

### Descripción:
Reinicia siempre el contenedor si se detiene. Si se detiene manualmente, solo se reiniciará cuando se reinicie el demonio Docker o cuando se reinicie manualmente el propio contenedor.

### Dockerfile (politica-always/Dockerfile):
```dockerfile
FROM alpine:latest
CMD echo "Contenedor con política ALWAYS - reiniciando..." && sleep 5
```

### Construcción:
```bash
cd politica-always
docker build -t test-always .
```

### Ejecución:
```bash
docker run -d --name contenedor-always --restart always test-always
```

ID: `b2c3d4e5f6g7`

### Verificación:
```bash
docker ps -a
```

Resultado inicial:
```
CONTAINER ID   IMAGE         COMMAND                  STATUS                          NAMES
b2c3d4e5f6g7   test-always   "/bin/sh -c 'echo Co…"   Up 3 seconds (Restarting)      contenedor-always
```

Contador de reinicios:
```bash
docker inspect contenedor-always --format='{{.RestartCount}}'
```

Resultado: `6`

### Observaciones:
- El contenedor se reinicia constantemente
- El contador aumenta cada vez que termina
- Continúa reiniciándose automáticamente
- Persiste después de reiniciar Docker

---

## Política: UNLESS-STOPPED

### Descripción:
Similar a always, excepto que cuando el contenedor se detiene manualmente, no se reinicia incluso después de reiniciar el demonio Docker.

### Dockerfile (politica-unless-stopped/Dockerfile):
```dockerfile
FROM alpine:latest
CMD echo "Contenedor con política UNLESS-STOPPED" && sleep 5
```

### Construcción:
```bash
cd politica-unless-stopped
docker build -t test-unless-stopped .
```

### Ejecución:
```bash
docker run -d --name contenedor-unless-stopped --restart unless-stopped test-unless-stopped
```

ID: `c3d4e5f6g7h8`

### Verificación:
```bash
docker ps -a
```

Resultado:
```
CONTAINER ID   IMAGE                  COMMAND                  STATUS                NAMES
c3d4e5f6g7h8   test-unless-stopped    "/bin/sh -c 'echo Co…"   Up 2 seconds         contenedor-unless-stopped
```

Contador de reinicios:
```bash
docker inspect contenedor-unless-stopped --format='{{.RestartCount}}'
```

Resultado: `4`

### Prueba de detención:
```bash
docker stop contenedor-unless-stopped
docker ps -a | findstr contenedor-unless-stopped
```

Resultado:
```
c3d4e5f6g7h8   test-unless-stopped   "/bin/sh -c 'echo Co…"   Exited (137) 10 seconds ago   contenedor-unless-stopped
```

### Observaciones:
- Se reinicia automáticamente durante la ejecución
- Al detenerlo manualmente, no vuelve a iniciar
- Permanece detenido incluso al reiniciar Docker
- Diferencia principal con always

---

## Política: ON-FAILURE

### Descripción:
Se reinicia únicamente cuando hay una falla en la ejecución del código que se manifiesta con un código diferente de 0. No se reinicia si el contenedor se detiene manualmente.

### Dockerfile (politica-on-failure/Dockerfile):
```dockerfile
FROM alpine:latest
CMD echo "Simulando un error..." && exit 1
```

### Construcción:
```bash
cd politica-on-failure
docker build -t test-on-failure .
```

### Ejecución:
```bash
docker run -d --name contenedor-on-failure --restart on-failure test-on-failure
```

ID: `d4e5f6g7h8i9`

### Verificación:
```bash
docker ps -a
```

Resultado:
```
CONTAINER ID   IMAGE              COMMAND                  STATUS                         NAMES
d4e5f6g7h8i9   test-on-failure    "/bin/sh -c 'echo Si…"   Restarting (1) 2 seconds ago   contenedor-on-failure
```

Código de salida:
```bash
docker inspect contenedor-on-failure --format='{{.State.ExitCode}}'
```

Resultado: `1`

Contador de reinicios:
```bash
docker inspect contenedor-on-failure --format='{{.RestartCount}}'
```

Resultado: `8`

Logs:
```bash
docker logs contenedor-on-failure --tail 10
```

Resultado:
```
Simulando un error...
Simulando un error...
Simulando un error...
Simulando un error...
Simulando un error...
```

### Observaciones:
- Termina con código de error 1
- Se reinicia por el fallo
- Contador aumenta continuamente
- Solo reinicia cuando hay error

---

## Tabla Comparativa

| Política | Reinicia en exit 0 | Reinicia en exit 1 | Reinicia después de docker stop | Reinicia después de reiniciar Docker |
|----------|-------------------|-------------------|--------------------------------|-------------------------------------|
| no | No | No | No | No |
| always | Sí | Sí | Sí | Sí |
| unless-stopped | Sí | Sí | No | No |
| on-failure | No | Sí | No | No |

---

## Comandos Utilizados

Ver política de reinicio:
```bash
docker inspect <nombre_contenedor> --format='{{.HostConfig.RestartPolicy.Name}}'
```

Ver contador de reinicios:
```bash
docker inspect <nombre_contenedor> --format='{{.RestartCount}}'
```

Ver código de salida:
```bash
docker inspect <nombre_contenedor> --format='{{.State.ExitCode}}'
```

Ver estado:
```bash
docker inspect <nombre_contenedor> --format='{{.State.Status}}'
```

Listar contenedores:
```bash
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Image}}"
```

Resultado:
```
NAMES                        STATUS                          IMAGE
contenedor-no                Exited (0) 5 minutes ago        test-no
contenedor-always            Up 15 seconds                   test-always
contenedor-unless-stopped    Exited (137) 2 minutes ago      test-unless-stopped
contenedor-on-failure        Restarting (1) 3 seconds ago    test-on-failure
```

---

## Casos de Uso

**no:**
- Contenedores de ejecución única
- Scripts de una sola vez
- Pruebas temporales

**always:**
- Bases de datos en producción
- Servidores web críticos
- Servicios 24/7

**unless-stopped:**
- Servicios de desarrollo
- Aplicaciones que se pausan manualmente
- Contenedores que no deben autoiniciarse

**on-failure:**
- Scripts que pueden fallar
- Aplicaciones con recuperación de errores
- Procesos que reinician solo en fallo

---

## Limpieza

Detener contenedores:
```bash
docker stop contenedor-no contenedor-always contenedor-unless-stopped contenedor-on-failure
```

Eliminar contenedores:
```bash
docker rm contenedor-no contenedor-always contenedor-unless-stopped contenedor-on-failure
```

Verificación:
```bash
docker ps -a
```

Resultado:
```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

Eliminar imágenes:
```bash
docker rmi test-no test-always test-unless-stopped test-on-failure
```

---

## Conclusiones

**Política no:** Configuración por defecto. Sin reinicios automáticos. Para tareas de una sola ejecución.

**Política always:** Máxima disponibilidad. Reinicia siempre, incluso después de reiniciar Docker. Para servicios críticos.

**Política unless-stopped:** Similar a always pero respeta detenciones manuales. No reinicia después de docker stop.

**Política on-failure:** Reinicia solo con errores (exit code diferente de 0). Eficiente para procesos que pueden fallar.

Diferencia clave always vs unless-stopped:
- always reinicia después de docker stop al reiniciar Docker
- unless-stopped no reinicia después de docker stop

Diferencia clave always vs on-failure:
- always reinicia sin importar el código de salida
- on-failure solo reinicia con errores

# Limitar uso de procesador
Limitar la cantidad de núcleos de CPU:
```
--cpus=<número de núcleos>
```

Asignar núcleos de CPU específicos:
```
--cpuset-cpus=<lista de núcleos>
```

**¿Como saber el numero de procesadores virtuales que tiene una máquina?**
```
C:\Users\ASUS TUF F15>wmic cpu get NumberOfLogicalProcessors
NumberOfLogicalProcessors
20
```
## Ejemplos
_Puedes copiar y ejecutar directamente cada uno de los comandos_

Limitar el uso de CPU a 1.5 núcleos
```
docker run -d --name server-nginx --cpus="1.5" nginx:alpine
```

Restringir el contenedor para que use los núcleos de CPU 0 a 2:
```
docker run -d --name server-nginx --cpuset-cpus="0-2" nginx:alpine
```

Restringir el contenedor para que use los núcleos de CPU 1 y 3:
```
docker run -d --name server-nginx --cpuset-cpus="1,3" nginx:alpine
```
```
CONTAINER ID   NAME           CPU %     MEM USAGE / LIMIT   MEM %     NET I/O         BLOCK I/O     PIDS
fcdf27ac96ac   servidor-web   0.01%     7.406MiB / 300MiB   2.47%     1.17kB / 126B   0B / 12.3kB   7
```

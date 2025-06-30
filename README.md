# Práctica Calificada 4

## Estudiante
Christian Giovanni Luna Jaramillo

## Correo Institucional
christian.luna.j@uni.pe

## Número de Proyecto
Proyecto 7

## Título del proyecto grupal
Observabilidad de clúster Kubernetes local (mini-monitoring)

## URL - Repositorio Grupal
https://github.com/grupo10-CC3S2/Proyecto7-PC4

## Descripción del rol
### Sprint 1
- Me encargué de crear scripts para recoger los logs de los Pods en un namespace específico y los eventos del Cluster (`log_collector.sh` y `log_collector.py`) para guardarlos en archivos locales además de implementar pruebas con marcas `xfail` o `skip` para casos donde no hay Pods disponibles `test_collector_log.py`

# Instrucciones de ejecución

## Sprint 1
Para esta primera parte del proyecto, lo que haremos será clonar el repositorio grupal sobre el que trabajamos, pero luego nos ubicaremos al último commit que realicé en ese repositorio, para verificar el funcionamiento de este, para ello lo primero que haremos será clonar el repositorio [Proyecto7-PC4](https://github.com/grupo10-CC3S2/Proyecto7-PC4) y entraremos en el repositorio
```bash
git clone https://github.com/grupo10-CC3S2/Proyecto7-PC4.git

cd Proyecto7-PC4
```
El hash del commit que para el sprint 1 usaremos, será `6cfcc8a` el cuál podemos verificar en la siguiente tabla, y en este caso será el último commit que tiene el repositorio
![Sprint1](Img/Sprint1/Commits-Sprint1.png)

Así que para entrar a ese momento del historial de commits, ejecutaremos el comando `git checkout`
```bash
git checkout 6cfcc8a
```
![6cfcc8a](Img/Sprint1/6cfcc8a.png)
Y con esto entraremos a un estado de detached, con lo cuál lo ideal es que cada cambio que hagamos, lo eliminemos a su estado base para no generar errores.
Ya estando en este momento del historial, procedemos a ejecutar los siguiente comandos
```bash
# Seguimos los pasos para levantar los Pods
# Construir imagen
docker build -t timeserver:latest app

# Desplegar pods
kubectl apply -f k8s/

# Comprobar que los pods corren
kubect get pods

# Para verificar que el servicio corre, ejecutar
```
Creamos el entorno virtual para instalar las dependencias que necesitamos
```bash
# Creando entorno virtual
python -m venv venv

# Entrar al entorno
# Powershell
.\venv\Scripts\activate
# Ubuntu
source venv/Scripts/activate

# Instalar requirements
pip install -r requirements.txt
```
### Probar scripts
Para esta parte, ejecutaremos todo en git bash, así que en caso se use Windows, ejecutar `git bash` pero en caso se use Ubuntu, dejar omitir ese comando.
#### log_collector.sh
```bash
bash scripts/log_collector.sh
```
![Img](Img/Sprint1/log_collector-sh.png)

Para probar el `log_collector.py` eliminamos la carpeta `logs` que se creó
```bash
rm -r logs/
```

#### log_collector.py
```bash
python scripts/log_collector.py
```
![Img](Img/Sprint1/log_collector-py.png)

Lugo de ejecutar el comando, no es necesario eliminar la carpeta logs, porque con la prueba unitaria se eliminará, pero por si acaso ejecutamos el comando
```bash
rm -r logs/
```

#### pytest.ini
Para realizar las pruebas, debemos de tener nuestra configuración `pytest.ini` para saber desde donde comenzar a buscar los test y tener listo el entorno para las pruebas unitarias
![Img](Img/Sprint1/pytest-ini.png)

#### test_collector_log.py
Como ya tenemos `pytest.ini` declarado, entonces solo ejecutaremos el comando `pytest`
```bash
pytest
```
![Img](Img/Sprint1/pytest.png)

Luego de realizar todos los pasos, elimamos los pods y la imagen y sería todo finalizado por mi parte para el **Sprint-1**
```bash
kubectl delete -f k8s
```
Luego de esperar unos segundos, ejecutar
```bash
docker image rm timeserver
```
Para volver al estado actual de `Proyecto7-PC4` solo ejecutamos el siguiente comando y ya no estaremos apuntando al commit seleccionado
```bash
git checkout main
```
Y con esto se acabaría toda mi constribución para el **Sprint-1**


## Sprint 2_3
La razon por la que creo las carpetas y este nombre como Sprint 2_3 es porque es un mismo archivo, en el cuál solo he modificado actualizar lo que pedían, la cuál visualizar las métricas que se obtiene con un script anterior, los pasos a seguir son los siguientes

```bash
# Clonar en caso no se tenga, sino entrar y solo realizar git pull
git clone https://github.com/grupo10-CC3S2/Proyecto7-PC4.git

cd Proyecto7-PC4
```
Luego seguimos el `README.md` para construir las imagenes y desplegar los pods
```bash
docker build -t timeserver:v1 app
docker build -t timeserver:v2 app

kubectl apply -f k8s/
```

Luego de tener todo listo, instalamos `metric server` para poder obtener las métricas de los pods, para esto seguimos las instrucciones del `README-visualizer.py`

## Instalar metrics-server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml


En caso ejecutemos la línea "kubectl top pods -n default" que indique algo como `API no disponible` lo que haremos será

### 1.- kubectl get pods -n kube-system

Y verificar que existe un pods llamado metric-server-<numero>-<codigo>

Si verificamos eso, guardamos el nombre del pods, y editaremos su archivo

```bash
kubectl edit deployment metrics-server -n kube-system
```
Esto nos abrirá un editor, y cuando haga esto, buscamos lo siguiente:
```bash
containers:
    - args:
        - --cert-dir=/tmp
        - --secure-port=10250
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
```
Cuando encontremos eso, lo que haremos será agregar la siguiente línea
```bash
        - --kubelet-insecure-tls
```
**Ojo** Tener cuidado con los espacios

Luego guardamos el editor, cerramos el archivo
### Mal
En caso hayamos editado mal nos aparecerá algo así cuando guardemos el archivo
```bash
error: deployments.apps "metrics-server" is invalid
```
Y nos abrirá otro editor

### Bien
En caso haberlo hecho bien, nos aparecerá este mensaje
```bash
deployment.apps/metrics-server edited
```

Ahora con esto, ya podemos ejecutar el comando

```bash
kubectl top pods -d default
```

Luego de verificar que podemos obtener las métricas, creamos el entorno virtual con las dependencias necesarias para ejecutar los scripts.
```bash
python -m venv venv

# Entrar al entrono
.\venv\Scripts\activate

# Instalar dependencias
pip install -r requirements.txt
```


Finalmente ejecutamos el archivo `metric_collector.py` ya que el `metric_visualizer.py` depende de el script anterior porque lee los datos que ese genera

```bash
python scripts/metric_collector/metric_collector.py
```

Luego de obtenida las métricas, procedemos a probar el archivo `metric_visualizer.py` que cree

```bash
python scripts/metric_collector/metric_visualizer.py
```

Ahora con esto ya vemos lo realizado para este sprint, el cuál nos muestra en la terminal las métricas para todos los pods existentes y también de los nodos, además de que realiza alertas en caso el nodo o pod supere el umbral esperado y tambien avise en caso el pod no está el estado `Ready`
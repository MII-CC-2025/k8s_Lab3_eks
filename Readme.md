# K8S - Elastic Kubernetes Service (EKS)


## Requisitos

### Instala AWS CLI

https://docs.aws.amazon.com/es_es/cli/latest/userguide/getting-started-install.html

```shell
$ aws --version 
aws-cli/2.15.52 Python/3.11.8 Windows/10 exe/AMD64
```

Identificaté en la consola mediante un perfil con:

```shell
$ aws configure
AWS Access Key ID [None]: ASIA...VPYJ
AWS Secret Access Key [None]: YgQ...ldrR
Default region name [None]: us-east-1
Default output format [None]:
```
```shell
$ aws configure set aws_session_token IQo...ts0=
```


### Instala kubectl

https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html


O utiliza alguno de los siguientes métodos alternativos:

https://kubernetes.io/docs/tasks/tools/




## Crea el cluster EKS

Utiliza la consola Web de AWS para crear el cluster: 
- Selecciona Elastic Kubernetes Service (EKS)
- Pulsa en Agregar Cluster y Crear
- Elige la opción: Custom configuration 
- Deshabilita la opción EKS Auto Mode (Nosotros crearemos los nodos)
- Inserta el nombre
- En Cluster IAM role elige el rol LabEKSClusterRole
- Deja el resto de opciones por defecto y pulsa siguiente
- Deja la VPC por defecto y elimina, si aparecen, las subredes dejando: "us-east-1a", "us-east-1b", "us-east-1c", "us-east-1d" y "us-east-1f".
- Establezce el "Acceso al punto de enlace del clúster" (Cluster endpoint access) como Público, deja el resto de opciones por defecto y pulsa siguiente
- Deja el resto de opciones por defecto, pulsando siguiente en las tres pantallas siguientes y pulsa en Crear.

La creación del cluster tardará un tiempo, una vez haya finalizado (aparecerá el estado del cluster como Activo).

A continuación, vamos a establecer los nodos de computación del cluster. Para ello, accedemos al cluster y pulsamos sobre Informática (Computation). Podemos establecer un grupo de nodos (máquinas virtuales) o un perfil Fargate para trabajar con computación servlerless.

En este caso, crearemos un perfil de nodos. Pulsamos en Agregar grupo de nodos, establecemos un nombre para el grupo de nodos, seleccionamos un rol (LabRol) y pulsamos en siguiente. 
Establecemos el tipo de AMI (Amazon Linux x86_64), podemos cambiar el tipo de instancia (si queremos ahorar en costes), podemos establecer el número deseado, mínimo y máximo de instancias y pulsamos en siguiente.
En las subredes pulsamos en siguiente y finalmante en siguiente y Crear.

Pasado un tiempo, veremos que aparecerán los nodos del clúster y su estado será "Preparado"

Para obtener las credenciales para conexión al cluster, ejecuta en la consola (acreditada para AWS) el siguiente comando. 

```
$ aws eks update-kubeconfig --region <region> --name <cluster> --alias <context>
Added new context <context> to .kube\config

```


Podemos probar si tenemos las credenciales y la conexión es correcta mediante el comando:

```
$ kubectl cluster-info 
Kubernetes control plane is running at https://<user_code>.gr7.us-east-1.eks.amazonaws.com
CoreDNS is running at https://<user_code>.gr7.us-east-1.eks.amazonaws.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

Además, podemos obtener información sobre las versiones del clientes y el servidor con el comando:

```
$ kubectl version -o=yaml  
clientVersion:
  buildDate: "2024-02-14T10:40:49Z"
  compiler: gc
  gitCommit: 4b8e819355d791d96b7e9d9efe4cbafae2311c88
  gitTreeState: clean
  gitVersion: v1.29.2
  goVersion: go1.21.7
  major: "1"
  minor: "29"
  platform: windows/amd64
kustomizeVersion: v5.0.4-0.20230601165947-6ce0bf390ce3
serverVersion:
  buildDate: "2024-04-30T23:53:58Z"
  compiler: gc
  gitCommit: 9c0e57823b31865d0ee095997d9e7e721ffdc77f
  gitTreeState: clean
  gitVersion: v1.29.4-eks-036c24b
  goVersion: go1.21.9
  major: "1"
  minor: 29+
  platform: linux/amd64
```
Si todo es correcto, veremos la versión del cliente y del servidor; si no tenemos las credenciales solo veremos la versión del cliente y un error de conexión al servidor.


Para más información sobre las principales opciones del comando kubectl consulta alguno de estos enlaces:

```
https://kubernetes.io/docs/reference/kubectl/cheatsheet/

https://jamesdefabia.github.io/docs/user-guide/kubectl/kubectl/

```
Por ejemplo, los siguientes comandos, permiten ver la configuración del fichero .kube/config, obtener los contextos (clusters) disponibles, ver el contexto actual y cambiar el contexto por defecto 

```
kubectl config view                         # Muestra fichero kube/config
kubectl config get-contexts                 # Muestra los contextos (clusters) disponibles
kubectl config current-context              # Muestra el contexto actual 
kubectl config use-context <cluster-name>   # Asigna el contexto por defecto a <cluster-name>
``` 
### Más información sobre el cluster

Versiones de las API de kubernetes disponibles:

```
$ kubectl api-versions
admissionregistration.k8s.io/v1
apiextensions.k8s.io/v1
apiregistration.k8s.io/v1
apps/v1
authentication.k8s.io/v1
authorization.k8s.io/v1
autoscaling/v1
autoscaling/v2
batch/v1
certificates.k8s.io/v1
coordination.k8s.io/v1
crd.k8s.amazonaws.com/v1alpha1
discovery.k8s.io/v1
events.k8s.io/v1
flowcontrol.apiserver.k8s.io/v1
flowcontrol.apiserver.k8s.io/v1beta3
networking.k8s.aws/v1alpha1
networking.k8s.io/v1
node.k8s.io/v1
policy/v1
rbac.authorization.k8s.io/v1
scheduling.k8s.io/v1
storage.k8s.io/v1
v1
vpcresources.k8s.aws/v1alpha1
vpcresources.k8s.aws/v1beta1
```

Recursos disponibles para crear en el clúster:

```
$ kubectl api-resources
NAME                              SHORTNAMES   APIVERSION                        NAMESPACED   KIND
bindings                                       v1                                true         Binding
componentstatuses                 cs           v1                                false        ComponentStatus
configmaps                        cm           v1                                true         ConfigMap
endpoints                         ep           v1                                true         Endpoints
events                            ev           v1                                true         Event
limitranges                       limits       v1                                true         LimitRange
namespaces                        ns           v1                                false        Namespace
nodes                             no           v1                                false        Node
persistentvolumeclaims            pvc          v1                                true         PersistentVolumeClaim
persistentvolumes                 pv           v1                                false        PersistentVolume
pods                              po           v1                                true         Pod
podtemplates                                   v1                                true         PodTemplate
replicationcontrollers            rc           v1                                true         ReplicationController
resourcequotas                    quota        v1                                true         ResourceQuota
secrets                                        v1                                true         Secret
serviceaccounts                   sa           v1                                true         ServiceAccount
services                          svc          v1                                true         Service
mutatingwebhookconfigurations                  admissionregistration.k8s.io/v1   false        MutatingWebhookConfiguration
validatingwebhookconfigurations                admissionregistration.k8s.io/v1   false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds     apiextensions.k8s.io/v1           false        CustomResourceDefinition
apiservices                                    apiregistration.k8s.io/v1         false        APIService
controllerrevisions                            apps/v1                           true         ControllerRevision
daemonsets                        ds           apps/v1                           true         DaemonSet
deployments                       deploy       apps/v1                           true         Deployment
replicasets                       rs           apps/v1                           true         ReplicaSet
statefulsets                      sts          apps/v1                           true         StatefulSet
selfsubjectreviews                             authentication.k8s.io/v1          false        SelfSubjectReview
tokenreviews                                   authentication.k8s.io/v1          false        TokenReview
localsubjectaccessreviews                      authorization.k8s.io/v1           true         LocalSubjectAccessReview
selfsubjectaccessreviews                       authorization.k8s.io/v1           false        SelfSubjectAccessReview
selfsubjectrulesreviews                        authorization.k8s.io/v1           false        SelfSubjectRulesReview
subjectaccessreviews                           authorization.k8s.io/v1           false        SubjectAccessReview
horizontalpodautoscalers          hpa          autoscaling/v2                    true         HorizontalPodAutoscaler
cronjobs                          cj           batch/v1                          true         CronJob
jobs                                           batch/v1                          true         Job
certificatesigningrequests        csr          certificates.k8s.io/v1            false        CertificateSigningRequest
leases                                         coordination.k8s.io/v1            true         Lease
eniconfigs                                     crd.k8s.amazonaws.com/v1alpha1    false        ENIConfig
endpointslices                                 discovery.k8s.io/v1               true         EndpointSlice
events                            ev           events.k8s.io/v1                  true         Event
flowschemas                                    flowcontrol.apiserver.k8s.io/v1   false        FlowSchema
prioritylevelconfigurations                    flowcontrol.apiserver.k8s.io/v1   false        PriorityLevelConfiguration
policyendpoints                                networking.k8s.aws/v1alpha1       true         PolicyEndpoint
ingressclasses                                 networking.k8s.io/v1              false        IngressClass
ingresses                         ing          networking.k8s.io/v1              true         Ingress
networkpolicies                   netpol       networking.k8s.io/v1              true         NetworkPolicy
runtimeclasses                                 node.k8s.io/v1                    false        RuntimeClass
poddisruptionbudgets              pdb          policy/v1                         true         PodDisruptionBudget
clusterrolebindings                            rbac.authorization.k8s.io/v1      false        ClusterRoleBinding
clusterroles                                   rbac.authorization.k8s.io/v1      false        ClusterRole
rolebindings                                   rbac.authorization.k8s.io/v1      true         RoleBinding
roles                                          rbac.authorization.k8s.io/v1      true         Role
priorityclasses                   pc           scheduling.k8s.io/v1              false        PriorityClass
csidrivers                                     storage.k8s.io/v1                 false        CSIDriver
csinodes                                       storage.k8s.io/v1                 false        CSINode
csistoragecapacities                           storage.k8s.io/v1                 true         CSIStorageCapacity
storageclasses                    sc           storage.k8s.io/v1                 false        StorageClass
volumeattachments                              storage.k8s.io/v1                 false        VolumeAttachment
cninodes                          cnd          vpcresources.k8s.aws/v1alpha1     false        CNINode
securitygrouppolicies             sgp          vpcresources.k8s.aws/v1beta1      true         SecurityGroupPolicy                                                warden.gke.io/v1                       false        Audit
```

Podemos solicitar información sobre un determinado recurso o de alguna de sus opciones:

```
$ kubectl explain --api-version=apps/v1 Deployment

$ kubectl explain --api-version=apps/v1 Deployment.metadata | more
$ kubectl explain --api-version=apps/v1 Deployment.spec.template | more
$ kubectl explain --api-version=apps/v1 Deployment.spec.template.spec.containers | more
```

## Consultar recursos del cluster

Podemos consutar los recursos del Clúster con: kubectl get

Por ejemplo, los espacios de nombre (ns: namespaces) con 

```
$ kubectl get ns   
NAME              STATUS   AGE
default           Active   95m
kube-node-lease   Active   95m
kube-public       Active   95m
kube-system       Active   95m
```
El espacio de nombre por defecto es default, todos los comandos se ejecutan sobre ese espacio de nombre a menos que se indique lo contrario. Así, para obtener los pods del espacio de nombre por defecto:

```
$ kubectl get pods      
No resources found in default namespace.
```
Podemos especifiar el espacio de nombre con: --namespace <espacio_nombres>

```
$ kubectl get pods --namespace kube-system 
NAME                           READY   STATUS    RESTARTS   AGE
aws-node-957h4                 2/2     Running   0          82m
aws-node-wzth2                 2/2     Running   0          80m
coredns-54d6f577c6-6b2k7       1/1     Running   0          81m
coredns-54d6f577c6-djhkj       1/1     Running   0          80m
eks-pod-identity-agent-wbs4r   1/1     Running   0          80m
eks-pod-identity-agent-xzjrt   1/1     Running   0          82m
kube-proxy-bvf6d               1/1     Running   0          82m
kube-proxy-zm92w               1/1     Running   0          80m
```

## Nodos

```
$ kubectl get nodes
NAME                            STATUS   ROLES    AGE    VERSION
ip-172-31-65-191.ec2.internal   Ready    <none>   148m   v1.29.3-eks-ae9a62a
ip-172-31-93-80.ec2.internal    Ready    <none>   146m   v1.29.3-eks-ae9a62a
```

## Métricas

https://docs.aws.amazon.com/es_es/eks/latest/userguide/metrics-server.html

Ejecuta:

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
Espera hasta que la implementación de las métricas esté activa:

```
kubectl get deployment metrics-server -n kube-system -w
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
metrics-server   0/1     1            0           29s
metrics-server   1/1     1            1           30s
```

Lista las métricas en cada nodo:

```
$ kubectl top nodes
NAME                            CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
ip-172-31-65-191.ec2.internal   31m          1%     521Mi           15%
ip-172-31-93-80.ec2.internal    30m          1%     475Mi           14%
```


## Crear el primer Pod

De forma imperativa, podemos crear un Pod con:

```
$ kubectl run nginx --image=nginx --port=80
pod/nginx created
```

También, podemos usar un fichero de manifiesto .yaml (por ejemplo, nginx-pod.yaml) con la configuración del Pod:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
```
y ejecutar, el comando imperativo:

```
$ kubectl create -f nginx-pod.yaml
pod/nginx created
```

O podemos ejecutarlo de forma declarativa con:

```
$ kubectl apply -f nginx-pod.yaml  
pod/nginx created
```

Ver los Pods:

```
$ kubectl get pods -o=wide
NAME    READY   STATUS    RESTARTS   AGE   IP          NODE             NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          29s   10.1.0.29   nodo1            <none>           <none>
```

Ver los logs de un Pod con kubectl logs <nombre_pod>

```
$ kubectl logs nginx
...
```

Ejecutar un comando en un Pod con kubectl exec <nombre_pod> -- <comando>  

```
$ kubectl exec -it nginx -- bash
root@nginx:/#

```

Obtener la descripción de un recurso con kubectl describe, en este caso de un Pod con:

```
kubectl describe pod nginx
...
```

Borrar recursos con kubectl delete

```
kubectl delete pod nginx
```
También podemos hacerlo usando el fichero yaml:

```
kubectl delete -f nginx-pod.yaml
```

## Exponer un Pod mediante un servicio de forma imperativa

### Servicio ClusterIP

```
$ kubectl expose pod nginx --port=80 --name=frontend
```

Ver los servicios (services):

```
$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
frontend     ClusterIP   10.100.18.125   <none>        80/TCP    2s
...
```

Ver los endpoints, a los que apunta un servicio:

```
kubectl get ep 
NAME         ENDPOINTS           AGE
frontend     10.1.0.30:80        103s
...
```

Ahora tenemos accesible el Pod (con IP 10.1.0.30) desde la IP del servicio (10.100.18.125).

Podemos entrar en cualquier Pod y comprobar que el servicio accede al Pod nginx.


### Servicio NodePort

```
$ kubectl expose pod nginx --port=80 --type=NodePort --name=frontend
```

Ver el servicio

```
$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
frontend     NodePort    10.108.59.115   <none>        80:31930/TCP   29s
```

Ahora el Pod es accesible desde la IP del Host mediante el puerto 31930.


```
$ ipconfig
Dirección IPv4. . . . . . . . . . . . . . : 192.168.18.34

$ curl 192.168.18.34:31930

```

### Servicio LoadBalancer

```
$ kubectl expose pod nginx --port=80 --type=LoadBalancer --name=frontend
NAME               TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/frontend   LoadBalancer   10.104.13.74   localhost     80:32523/TCP   2m6s
```
Ahora el Pod es accesible mediante la IP externa (EXTERNAL-IP) asignada al balanceador.

### Mostrar recursos

Todos los recursos (all):

```
kubectl get all    
NAME        READY   STATUS    RESTARTS   AGE
pod/nginx   1/1     Running   0          136m

NAME               TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/frontend   LoadBalancer   10.104.13.74   localhost     80:32523/TCP   2m6s

```

Por etiqueta (-l), en este caso run=nginx:

```
kubectl get all -l run=nginx     
NAME        READY   STATUS    RESTARTS   AGE
pod/nginx   1/1     Running   0          136m

NAME               TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/frontend   LoadBalancer   10.104.13.74   localhost     80:32523/TCP   2m6s

```

### Eliminar recursos por etiquetas

```
$ kubectl delete all -l run=nginx 
pod "nginx" deleted
service "frontend" deleted
```

## Implementar Aplicación con Go

https://go.dev/doc/tutorial/getting-started

Crear la carpeta app para el proyecto y habilitar las dependencias:

```
$ mkdir app
$ cd app

$ go mod init example/hello
```

Crear un fichero hello.go con el siguiente contenido:

```go

package main

import (
        "fmt"
        "log"
        "net/http"
        "os"
)

func handler(w http.ResponseWriter, r *http.Request) {

        hostname, err := os.Hostname()
        if err != nil {
                fmt.Fprintln(w, "Error:", err)
        } else {
                fmt.Fprintln(w, "Hello from", hostname, "(version 2024)")
        }
}

func main() {
        http.HandleFunc("/", handler)
        log.Println("Go Hello is listening on port 8080")
        log.Fatal(http.ListenAndServe(":8080", nil))
}

```

Comprobemos el funcionamiento en local:

```
$ go run hello.go
2024/05/16 09:15:38 Go Hello is listening on port 8080
```


## Contenerizar la App

Dockerfile

```dockerfile
FROM golang:1.22.3-alpine AS build
WORKDIR /src/
RUN go mod init example/hello
COPY app/hello.go /src/
RUN go mod tidy
RUN CGO_ENABLED=0 go build -o /bin/hello

FROM scratch
EXPOSE 8080
COPY --from=build /bin/hello /bin/hello
ENTRYPOINT ["/bin/hello"]
```

Construir la imagen:

```shell 
docker build -t jluisalvarez/go_hello:2024 .
```

Comprobar funcionamiento de la imagen:

```shell 
docker run -d -p 8080:8080 --name hello_go jluisalvarez/go_hello:2024   
```

Subir la imagen a Docker Hub:

```shell

docker push jluisalvarez/go_hello:2024

```

## Crear Deployment en Kubernetes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-deployment
  labels:
    app: hello
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
      - name: go-hello
        image: jluisalvarez/go_hello:2024
        ports:
        - containerPort: 8080
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
```

Despliega en Kubernetes:

```shell
kubectl apply -f k8s/hello_deploy.yaml
```

## Crear servicio

Crear el fichero hello_service.yaml para exponer el deployment en un servicio tipo load balancer.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-service
spec:
  selector:
    app: hello
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
  type: LoadBalancer
```

Aplica el manifiesto con:

```shell
$ kubectl apply -f k8s/hello_service.yaml
```

## Escalar la aplicación

Podemos escalar la aplicación de forma manual con:

A patir del fichero

```
kubectl scale --replicas=3 -f k8s/hello_deploy.yaml
```

El recurso concreto:

```
kubectl scale --replicas=1 deployment/hello-deployment
```

### Autoescalado

También, podemos definir un escalado automático en función de parámetros como el uso de la CPU. Para ello,
debemos tener disponibles en nuestro clúster las métricas. 

```
kubectl autoscale deployment hello-deployment --max=3 --min=1 --cpu-percent=50
```

#### NOTA

Simula peticiones en Linux:

```
for ((i=1;i<=10000000;i++)); do   curl --header "Connection: keep-alive" "http://localhost:8080/"; done
```
Simula peticiones en Windows:

```
@echo off
for /L %%i in (1,1,10000000) do (
curl --header "Connection: keep-alive" "http://localhost:8080/"
)
```

## Desplegar una nueva versión

En kubernetes un Deployment nos permite definir la estrategia para publicar las actualizaciones, lo que se conoce como rollout. En la estrategia por defecto, RollingUpdate, la actualización sigue un enfoque gradual.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 0
  selector:
    ...
  ```

Etiqueta la versión actual

```
kubectl annotate deployment/hello-deployment kubernetes.io/change-cause="version 2024"
```

Provoca un cambio de imagen, imperativo: 

```
kubectl set image deployments/hello-deployment go-hello=jluisalvarez/go_hello:2025
```

(Esto también, podría haberse realizado modificando el fichero yaml y aplicando o editando directamente el deployment)

Ver versiones
```
kubectl rollout history deployment/hello-deployment
```

Anotaciones del motivo del cambio
```
kubectl annotate deployment/hello-deployment kubernetes.io/change-cause="New version 2025"
```

Comprobar estado de la actualización:
```
kubectl rollout status deployments/hello-deployment
```

Deshacer actualización:
```
kubectl rollout undo deployments/hello-deployment
```
```
kubectl rollout undo --to-revision=2 deployments/hello-deployment
```

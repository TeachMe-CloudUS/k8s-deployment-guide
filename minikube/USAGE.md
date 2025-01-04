# Guía de Despliegue con Minikube

Este documento proporciona instrucciones sobre cómo lanzar los archivos de Kubernetes ubicados en la carpeta `minikube`.

## Prerrequisitos

Antes de comenzar, asegúrate de tener instalados los siguientes componentes:

- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

## Pasos para lanzar los archivos de Kubernetes

1. **Iniciar Minikube**:
   Abre una terminal y ejecuta el siguiente comando para iniciar Minikube:

```sh
minikube start
```

2. **Aplicar los archivos de Kubernetes**:
   Ejecuta el siguiente comando para aplicar todos los archivos de configuración de Kubernetes en la carpeta:

```sh
kubectl apply -f configMap.yml
kubectl apply -f infraestructure.yml
kubectl apply -f services.yml
```

3. **Verificar los recursos desplegados**:
   Puedes verificar que los recursos se hayan creado correctamente con los siguientes comandos:

```sh
kubectl get all
```

## Detener Minikube

Una vez que hayas terminado de trabajar con Minikube, puedes detenerlo con el siguiente comando:

```sh
kubectl delete -f services.yml
kubectl delete -f infraestructure.yml
kubectl delete -f configMap.yml
minikube stop
```

## Recursos adicionales

- [Documentación de Minikube](https://minikube.sigs.k8s.io/docs/)
- [Documentación de Kubernetes](https://kubernetes.io/docs/)

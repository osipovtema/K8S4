# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 2»

### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к двум приложениям снаружи кластера по разным путям.

------

### Задание 1. Создать Deployment приложений backend и frontend

1. Создать Deployment приложения _frontend_ из образа nginx с количеством реплик 3 шт.  
- [deploy_front.yaml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-1-5/deploy_front.yaml)
2. Создать Deployment приложения _backend_ из образа multitool.  
- [deploy_back.yaml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-1-5/deploy_back.yaml)
3. Добавить Service, которые обеспечат доступ к обоим приложениям внутри кластера.  
- [service_front.yaml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-1-5/service_front.yaml)
- [service_back.yaml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-1-5/service_back.yaml)
4. Продемонстрировать, что приложения видят друг друга с помощью Service.
```
yura@Skynet kubernetes % kubectl run mycurlpod --image=curlimages/curl -i --tty --rm -- sh
If you don't see a command prompt, try pressing enter.
~ $ curl frontend-deployment
curl: (6) Could not resolve host: frontend-deployment
~ $ curl frontend-service
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

```
~ $ curl backend-service
WBITT Network MultiTool (with NGINX) - backend-deployment-bd94f76bd-std6k - 10.1.207.182 - HTTP: 80 , HTTPS: 443 . (Formerly praqma/network-multitool)
```
5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

------

### Задание 2. Создать Ingress и обеспечить доступ к приложениям снаружи кластера

1. Включить Ingress-controller в MicroK8S.
```
yura@Skynet kubernetes % helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
yura@Skynet kubernetes % helm repo update
yura@Skynet kubernetes % helm install ingress-nginx ingress-nginx/ingress-nginx
NAME: ingress-nginx
LAST DEPLOYED: Sun Nov  5 15:23:02 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace default get services -o wide -w ingress-nginx-controller'

An example Ingress that makes use of the controller:
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: example
    namespace: foo
  spec:
    ingressClassName: nginx
    rules:
      - host: www.example.com
        http:
          paths:
            - pathType: Prefix
              backend:
                service:
                  name: exampleService
                  port:
                    number: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
      - hosts:
        - www.example.com
        secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls

```

2. Создать Ingress, обеспечивающий доступ снаружи по IP-адресу кластера MicroK8S так, чтобы при запросе только по адресу открывался _frontend_ а при добавлении /api - _backend_.
- [ingress.yaml](https://github.com/kibernetiq/netology_k8s/blob/kuber-hw-1-5/ingress.yaml)
3. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.
- Кластер MicroK8S развернут на виртуальной машине, в данном случае ingress-nginx-controller с типом LoadBalancer не сможет получить EXTERNAL-IP
```
yura@Skynet kubernetes % kubectl get svc
NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
kubernetes                           ClusterIP      10.152.183.1     <none>        443/TCP                      12d
frontend-service                     ClusterIP      10.152.183.208   <none>        80/TCP                       85m
backend-service                      ClusterIP      10.152.183.157   <none>        80/TCP                       85m
ingress-nginx-controller-admission   ClusterIP      10.152.183.65    <none>        443/TCP                      19m
ingress-nginx-controller             LoadBalancer   10.152.183.179   <pending>     80:32471/TCP,443:31845/TCP   19m
```
4. Предоставить манифесты и скриншоты или вывод команды п.2.

------
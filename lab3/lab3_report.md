University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)  
Year: 2023/2024  
Group: K4110c  
Author: Soldatov Andrey Fedorovich    
Lab: Lab3  
Date of create: 20.11.2023  
Date of finished: 25.11.2023  

## Лабораторная работа №3 "Сертификаты и "секреты" в Minikube, безопасное хранение данных."

### Цель работы  
Познакомиться с сертификатами и "секретами" в Minikube, правилами безопасного хранения данных в Minikube.
### Ход работы
Запускаем minikube  
```
minikube start
```

После запуска создаем ConfigMap с переменными REACT_APP_USERNAME, REACT_APP_COMPANY_NAME и устанавливаем им значения.  


![img](img/1.png)  


Далее нужно было создать replicaset с 2 репликами и используя созданный ранее ConfigMap передать значения в поды нашего репликасета.  
![img](img/2.png)  

Запустим команду:  
```minikube dashboard```  
Откроется локальный веб интерфейс нашего кубера, перейдем во вкладку Pods, далее выберем из двух реплик один под и посмотрим на их environment variables.  
![img](img/6.png)  
Мы убедились, что значения из ConfigMap были подтянуты для наших подов.  
Идем дальше, следующее, что требовалось сделать, это сгенерировать серты и импортировать их в миникуб.  
Используем openssl, запустим команды: 
```
openssl genrsa -out cert/domain.key 2048
openssl req -key cert/domain.key -new -out cert/domain.csr
openssl x509 -signkey cert/domain.key -in cert/domain.csr -req -days 365 -out cert/domain.crt
```  
![img](img/7.png)  
Установим рандомное значение FQDN - secretdomainlocal.test
После этого используем команду для создания секретов и импорта в миникуб.
```
kubectl create secret tls tls-secret -o yaml --key=cert/domain.key --cert=cert/domain.crt > secret.yml
```
![img](img/3.png)  
В файле secret.yaml можно установить свои значения для сертификата и ключа.  
Далее проверим подключен ли у нас ingress контроллер с помощью команды: 
```
minikube addons enable list
```
В появившемся списке видим, что дополнение ingress у нас уже установлено и команду ```minikube addons enable ingress``` можно не запускать, мы это сделали в прошлый раз.  
Далее запускаем наше сервис с помощью команды ```kubectl apply -f service.yaml```
![img](img/4.png)  
Проверим запущен ли наш сервис ```kubectl get services```  
![img](img/8.png)  
Видим, что сервис запущен.  
Далее запускаем наш ingress манифест с помощью команды ```kubectl apply -f lab3-ingress.yaml```  
![img](img/5.png)  
Проверим, запустился или нет ```kubectl get ingress```
![img](img/9.png)  
Далее используем команду ```minikube tunnel``` для того, чтобы подключиться к нашему сервису.  
![img](img/10.png)  
Далее перейдем по адресу https://secretdomainlocal.test  
![img](img/11.png)  
Сделав переход на по этому адресу видим, что наще приложение работает, переменные из ConfigMap были подтянуты.  
Обновим страничку и увидим, что запросы перераспределяются между подами.  
![img](img/12.png)   

Далее представлены скрины информации по сертификату.  
![img](img/13.png)
![img](img/14.png)
![img](img/15.png)


Схема  
![img](img/schema.png)  

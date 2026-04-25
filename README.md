# Дипломный практикум в Yandex.Cloud
## Ссылка на манифесты Terraform и Ansible Role - https://github.com/hawk0774/1-infra-repo
1. Через терраформ поднимаются 6 виртуальных машин - 3 мастер ноды (в разных зонах доступности) и 3 воркер ноды, используется S3 для хранения State файла (готовим все для него в отдельном каталоге), ставится ALB для балансировки в приложения : APP, Grafana, Monitoring, ArgoCD. Ставится external NLB балансировщик (internal не работает в разных подсетях и зонах доступности) как аналог kube-vip для балансировки kube-api. Настраиваются сети, создаются группы безопасности, с ограничением доступа к нодам кластера только изнутри, подключения снаружи по ssh ограничены моим белым статическим ip, healscheckами яндекса, запрещен анонимный доступ кроме хелсчеков для kube-api, также генерируется динамический инвентори.
2. Через Ansible производится установка с помощью kubeadm, установка пререквизитов (пакеты для работы куба, настройки ядра для портфорвардинга и оверлейфс,копирование конфигов containerd и т.д.), далее инициализация и присоединение других мастеров и воркеров, с копированием куб конфига на мой домашний хост.Далее производится установка и настройка CNI плагина CALICO, установка istio и вместе с ним istio ingress gateway, он нам пригодится в качестве gatweway controller при работе gateway api (таска для установки нужных CRDшек прилагается), в общем все по-взрослому. Далее ставится ARGOCD, который будет отвечать у нас за континиум деплоймент. Отключаем анонимный доступ в кластер. При установке арго мы сразу натравливаем его на наш репозиторий https://github.com/hawk0774/3-gitops-repo, котором мы поговрим позже, единственное о нем сейчас можно упомянуть в контексете установки kube-state-monitoring (Grafana и prometheus, а так же конфиги для нашего приложения и связки gateway api с istio)
3. Как все происходит с приложением. Я пушу по тегу со своего локального ПК в репозиторий https://github.com/hawk0774/2-app-repo любое изменение. На это реагирует GithubAction. Он коннектится к реджистри, созданное в YC собирает контейнер с лейблом по тегу и кладет его в реджистри, плюс параллельно идет в репу https://github.com/hawk0774/3-gitops-repo в manifests/test-app/deployment.yaml и правит там тег образа, в это время ARGOCD видит изменения, и применяет их самостоятельно.
4. Как все происходит с мониторингом. Мы натравливаем ArgoCD, он ставит мониторинг.
5. Как все происходит с gateway api istio и в целом маршрутизацией. Схема такая - так как жадный яндекс не тает в metallb, говорит ставь кубер как сервис и там будет тебе оператор с балансировкой, то пришлось использовать node port для istio ingress gateway. Схема такая мы запрашиваем в браузере http://81.26.178.3/app или http://81.26.178.3/argocd или http://81.26.178.3/grafana или http://81.26.178.3/prometheus/  то есть идем на ALB яндекса --> Node port istio ingress gateway --> gateway --> HTTPROUTE --> endpoint service --> pod. С правилами можно ознакомится здесь - https://github.com/hawk0774/3-gitops-repo/tree/main/manifests/istio.

#Таким образом, мы имеем три репозитория:
 ##https://github.com/hawk0774/1-infra-repo
 ##https://github.com/hawk0774/2-app-repo
 ##https://github.com/hawk0774/3-gitops-repo

 Эндпоинты для подключения к:
 Grafana - http://81.26.178.3/grafana (login - admin, пароль - f47iev6imyvsckNr)
 ArgoCD - http://81.26.178.3/argocd (login - admin, пароль - f47iev6imyvsckNr)
 APP - http://81.26.178.3/app
 Prometheus - http://81.26.178.3/prometheus/

# Ставим контейнер реджистри

 ![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_13.png)
   
# Запускаем terraform apply  --auto-approve

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_2.png)

# Видим подтянутый state

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_4.png)

# Видим наш каталог и динамичсекий инвентори

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_6.png)

#Смотрим что создалось в консоли 

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_3.png)

# Смотрим на vm

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_26.png)

# Правила безопасности

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_24.png)

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_25.png)

# Запускаем playbook

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_8.png)

# Запрещаем анонимный доступ

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_27.png)

# Смотрим на ALB и NLB, так же в созданный реджистри

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_5.png)
![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_7.png)
![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_9.png)

# Щупаем кластер со своего ПК

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_10.png)

# Смотрим насколько успешно натравлен на репозиторий ARGOCD

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_11.png)

# Смотрим на приложение

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_1.png)

# Вносим изменение в статическую страницу и добавляем упоминание о теге 2.0.0, затем пушим с этим же тэгом и видим цепочку, которая приведет к деплою новой версии.

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_17.png)

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_18.png)

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_19.png)

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_23.png)

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_20.png)

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_21.png)

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_22.png)

# Смотрим на grafana и prometheus

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_12.png)

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_14.png)

![alt text](https://raw.githubusercontent.com/hawk0774/diplom-devops/main/Screenshot_15.png)




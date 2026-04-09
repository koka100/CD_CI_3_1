# Отчёт по лабораторной работе №3
## Вариант 20: Apache Spark кластер

**ФИО:** Константин Бойко  
**Группа:** АДЭУ-221
**Вариант:** 20

### Цель работы
Освоить оркестрацию контейнеров в Kubernetes, развернуть кластер Apache Spark (master + worker), настроить масштабирование через Deployment и сетевую доступность через Service.

### Ход выполнения

#### 1. Deployment для Spark Master
[вставить листинг spark-master-deployment.yaml]
- `image: spark:3.5.0` — официальный образ Spark
- `SPARK_MODE=master` — режим работы
-Ports: 7077 (RPC), 8080 (Web UI)

#### 2. Deployment для Spark Worker
[вставить листинг spark-worker-deployment.yaml]
- `SPARK_MASTER_URL=spark://spark-master-service:7077` — подключение к master
- `replicas: 1` — один worker

#### 3. Service для Master
[вставить листинг spark-master-service.yaml]
- `type: NodePort`
- `nodePort: 30080` — доступ к веб-интерфейсу

### Скриншоты
kubectl get pods <img width="632" height="90" alt="image" src="https://github.com/user-attachments/assets/2ad243c9-3762-4f83-b516-bf0d65e3e432" />



kubectl get services <img width="717" height="167" alt="image" src="https://github.com/user-attachments/assets/f41fa980-84e2-4af3-8232-b63f90eff557" />


Spark Web UI <img width="1206" height="602" alt="image" src="https://github.com/user-attachments/assets/b4aac8b4-b2a8-4ec5-96db-3f37f1790e00" />


### Выводы
- Kubernetes упрощает развёртывание распределённых систем (Spark кластер).
- Deployment обеспечивает автоматическое поддержание нужного количества подов.
- Service позволяет компонентам взаимодействовать по имени сервиса.
- Трудности: 
NodePort `30080` не слушается на хост‑машине (проверено через `netstat -tulnp | grep 30080`).  
Однако веб‑интерфейс Spark Master доступен внутри кластера:

```bash
kubectl exec -it spark-master-d8456fdd7-m77t2 -- curl -v http://localhost:8080
```

Для проверки в браузере использован `kubectl port-forward`:

```bash
kubectl port-forward service/spark-master-service 8080:8080
```

В браузере открыт адрес: `http://localhost:8080` — отображается интерфейс Spark Master.

Это доказывает, что кластер Apache Spark развёрнут и работает, а проблема в недостатке проброса NodePort в MicroK8s на виртуалке, а не в самом приложении.

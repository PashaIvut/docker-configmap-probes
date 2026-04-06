# ЛР: Продвинутые сценарии работы с ConfigMap, probes и ресурсами в Kubernetes
## Среда выполнения - Killercoda.
## Часть 1. Обновление ConfigMap и применение новой конфигурации
### Задание 1. 
Cоздадим **namespace** и сделаем его текущим.  
<img width="967" height="131" alt="image" src="https://github.com/user-attachments/assets/f79a46cd-9188-49f4-bb63-da6e104262d7" />

"Создадим" ConfigMap со старым содержимым.
<img width="500" height="219" alt="image" src="https://github.com/user-attachments/assets/9228e4e5-77e1-415a-934d-de4af74b1fa7" />  

И применим.  
<img width="818" height="47" alt="image" src="https://github.com/user-attachments/assets/4c967092-4be8-4e3c-8568-73a25f923c43" />  

Создадим и применим Deployment.  
<img width="530" height="656" alt="image" src="https://github.com/user-attachments/assets/5149ad39-c960-486e-ac70-554e21943b31" />  

Создадим и применим Service.  
<img width="460" height="361" alt="image" src="https://github.com/user-attachments/assets/07781986-5e18-47d3-af60-e8557f96b66f" />  

Получаем номер порта, назначенный сервису web-service.  
<img width="915" height="75" alt="image" src="https://github.com/user-attachments/assets/384d0f4a-6fe4-4896-b0d3-101f74552c8a" />  

Проверяем текущее состояние страницы с помощью **curl http://localhost:31376**.  
<img width="1456" height="471" alt="image" src="https://github.com/user-attachments/assets/6fdfb95c-b003-4999-9ac5-bd87d4fbe488" />  

Далее обновим ConfigMap.  
<img width="500" height="232" alt="image" src="https://github.com/user-attachments/assets/92f4dda8-60f7-407a-8325-fc28ea04165b" />  
 
Проверим. Все еще OLD!!!  
<img width="1439" height="604" alt="image" src="https://github.com/user-attachments/assets/3012e993-648f-414e-8da6-317231de9a45" />  

После обновления ConfigMap страница по-прежнему показывает OLD. Это демонстрирует, что изменение ConfigMap не применяется автоматически к работающим подам.

Перезапустим поды с помощью **kubectl rollout restart deployment web**. 
Разбор:
-- restart - действие — перезапустить поды;
-- deployment - тип ресурса;
-- web - имя deployment'а.
<img width="848" height="61" alt="image" src="https://github.com/user-attachments/assets/ecc73e9b-69b5-4c88-8c62-8bdc568bd1fe" />  

Состояние обновилось.  
<img width="1447" height="595" alt="image" src="https://github.com/user-attachments/assets/cd910b55-5ba8-4ad1-966d-759980376796" />  

## Часть 2. Startup probe и медленный старт приложения
Создадим Deployment без startupProbe.  
<img width="556" height="818" alt="image" src="https://github.com/user-attachments/assets/434518fa-cf20-4164-ae76-874d0afef101" />  

Применим.  
<img width="898" height="41" alt="image" src="https://github.com/user-attachments/assets/cac237cb-8b83-40a6-a8be-0c11430f9c1c" />  

Под работает.  
<img width="782" height="106" alt="image" src="https://github.com/user-attachments/assets/4cc1872f-e50f-499c-b18f-e5e9629a6efa" />  

Добавим задержку старта и применим снова.
<img width="620" height="847" alt="image" src="https://github.com/user-attachments/assets/fcd1f29d-c572-4aff-986e-5d5f70753bb0" />  

Наблюдаем проблемы.  
<img width="895" height="318" alt="image" src="https://github.com/user-attachments/assets/ccaea272-346d-4d13-85fb-f52cba76cff9" />
Когда Deployment web-slow применяется с postStart: sleep 60, контейнер nginx запускается, но не отвечает на HTTP-запросы в течение 60 секунд, потому что команда sleep блокирует запуск основного процесса nginx.  
Это приводит к проблемам с пробами. LivenessProbe проверяет, жив ли контейнер, и если проверка не проходит, перезапускает контейнер. В данном случае livenessProbe начинает проверять порт 80 через 0 секунд после запуска, но nginx не отвечает, поэтому проверка падает. После трёх неудачных проверок подряд убивает контейнер и перезапускает его, увеличивая счётчик RESTARTS.  
ReadinessProbe проверяет, готов ли под принимать трафик. Если readinessProbe не проходит, под получает статус NotReady, и Service не направляет на него запросы. В отличие от livenessProbe, readinessProbe не перезапускает контейнер, но под остаётся недоступным для трафика. В данном случае readinessProbe тоже не проходит, потому что nginx не отвечает, поэтому под всё время остаётся в статусе 0/1 Ready.













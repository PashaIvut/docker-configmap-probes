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











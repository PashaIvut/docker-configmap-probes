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
<img width="630" height="847" alt="image" src="https://github.com/user-attachments/assets/3e96e660-e73f-4851-afc2-c1da85002169" />  

Наблюдаем проблемы.  
<img width="895" height="318" alt="image" src="https://github.com/user-attachments/assets/ccaea272-346d-4d13-85fb-f52cba76cff9" />
Когда Deployment web-slow применяется с задержкой, контейнер nginx запускается, но не отвечает на HTTP-запросы в течение 60 секунд.  
Это приводит к проблемам с пробами. LivenessProbe проверяет, жив ли контейнер, и если проверка не проходит, перезапускает контейнер. В данном случае livenessProbe начинает проверять порт 80 через 0 секунд после запуска, но nginx не отвечает, поэтому проверка падает. После трёх неудачных проверок подряд убивает контейнер и перезапускает его, увеличивая счётчик RESTARTS.  
ReadinessProbe проверяет, готов ли под принимать трафик. Если readinessProbe не проходит, под получает статус NotReady, и Service не направляет на него запросы. ReadinessProbe не перезапускает контейнер, но под остаётся недоступным для трафика. В данном случае readinessProbe тоже не проходит, потому что nginx не отвечает, поэтому под всё время остаётся в статусе 0/1 Ready.

Добавим StartupProbe и применим. 
<img width="884" height="952" alt="image" src="https://github.com/user-attachments/assets/9c3f0916-ce5c-45fc-8652-9912fa7c2cb4" />  

Новый под появился без проблем, странностей не наблюдаем.  
<img width="777" height="109" alt="image" src="https://github.com/user-attachments/assets/51f135f2-ff50-4b02-bc1c-fc42c6d8a96d" />  
Новый под (web-slow-5fdf9655dc-6bx4f) с startupProbe запускается успешно. Он не перезапускается, потому что startupProbe даёт ему достаточно времени на старт, и он спокойно переживает задержку в 60 секунд.  

Финальное состояние пода.  
<img width="1477" height="747" alt="image" src="https://github.com/user-attachments/assets/04ca24fd-5ca2-41c4-9fac-d2ccbed57ff9" />  

## Часть 3. Ошибка liveness probe и перезапуски контейнера
Создадим файл с ошибочной livenessProbe.  
<img width="585" height="711" alt="image" src="https://github.com/user-attachments/assets/dcbc1757-cd31-4c36-b52e-bf36007c41a9" />  
<img width="921" height="51" alt="image" src="https://github.com/user-attachments/assets/926b69cc-e7d5-4b5f-99b4-37dfc7df33c2" />  

Наблюдаем перезапуски.  
<img width="817" height="126" alt="image" src="https://github.com/user-attachments/assets/240f33ce-3f59-4716-9ebf-5360b11d3e9c" />  

В логах беда.  
<img width="1475" height="702" alt="image" src="https://github.com/user-attachments/assets/aa74c9fc-e592-458b-987f-94d36c5f2f4a" />  

Вернем правильный путь.  
<img width="666" height="691" alt="image" src="https://github.com/user-attachments/assets/e673ed06-0963-48e2-871c-af1d348c7a4c" />  

Все восстановилось, все работает.  
<img width="908" height="166" alt="image" src="https://github.com/user-attachments/assets/3a95e59a-ee0c-449a-9ae6-98af8c562326" />  

## Часть 4. Слишком маленький memory limit и OOMKilled
Создадим Deployment с маленьким memory limit.
<img width="811" height="677" alt="image" src="https://github.com/user-attachments/assets/27ece1cc-2873-4ace-a88d-96de6cdb7453" />  
polinux/stress - это Docker-образ с программой stress — инструментом для создания искусственной нагрузки на систему.  
--vm 1 — запускает 1 процесс, потребляющий память; --vm-bytes 150M — выделяет 150 MB памяти; --vm-hang 1 — зависает на 1 секунду, чтобы процесс не завершался сразу.  

Наблюдаем перегрузки.  
<img width="912" height="285" alt="image" src="https://github.com/user-attachments/assets/e44114b6-511c-49be-9833-cb4db8747eaf" />  
<img width="678" height="220" alt="image" src="https://github.com/user-attachments/assets/bd4d3a9e-3781-40c9-8ff6-d5cd68d2ba20" />  

Увеличим лимит.  
<img width="780" height="670" alt="image" src="https://github.com/user-attachments/assets/e4a9ac71-70e1-4af7-9b9e-bdf3b7ff6e62" />  

Запуск прошел без проблем.  
<img width="830" height="168" alt="image" src="https://github.com/user-attachments/assets/cbfc01cd-6721-457d-854b-ec212bf001f6" />  

## Часть 5. Слишком низкий CPU limit и деградация работы
Создадим Deployment с низким CPU limit.  
<img width="551" height="664" alt="image" src="https://github.com/user-attachments/assets/6aad2c21-cb2c-476d-b3cd-010feb27f4c0" />

При CPU limit 50m и отсутствии нагрузки время ответа составляет 3-7 мс.

Создадим отдельный сервис, узнаем порт.

При CPU limit 50m и отсутствии дополнительной нагрузки nginx успешно обрабатывает запросы за 3-7 миллисекунд. Это нормальная работа контейнера.
<img width="1238" height="681" alt="image" src="https://github.com/user-attachments/assets/dfd5e22c-38cd-4da8-b9b8-0d34961a11f3" />

Теперь пустим нагрузку в нескольких терминалах.
<img width="1220" height="140" alt="image" src="https://github.com/user-attachments/assets/4c83e9fe-36a2-40fb-9088-d789f94bbccb" />
Время обработки увеличилось.

Уберем ограничения по CPU.  
<img width="580" height="577" alt="image" src="https://github.com/user-attachments/assets/ccd556d4-d3fc-4e48-ae88-47055a26d0b3" /> 

Время ответа составляет 3-5 мс, заметно.  
<img width="1214" height="396" alt="image" src="https://github.com/user-attachments/assets/b51cbc32-3f24-4028-b56d-c4e105b16ecb" />  
Без ограничения CPU контейнер работает быстрее, потому что может использовать всё доступное ядро.
CPU limit вызывает throttling (ограничение доступа к процессору), а не остановку контейнера. Контейнер продолжает работать, но получает меньше процессорного времени, из-за чего операции выполняются медленнее.

## Часть 6. Pod Pending из-за завышенных requests
Создадим Deployment с завышенными requests.  
<img width="540" height="645" alt="image" src="https://github.com/user-attachments/assets/7568482e-fc96-45a3-a4b0-04b279713767" />  

Проверим под. Он в статусе Pending.  
<img width="893" height="235" alt="image" src="https://github.com/user-attachments/assets/87704572-0d50-4386-94cb-7cdacabeedba" />  
Scheduler не может разместить под, потому что нет ноды с таким количеством свободных ресурсов (100 ядер и 500GB памяти).

В описании пода предупреждение.  
<img width="1454" height="121" alt="image" src="https://github.com/user-attachments/assets/6294b03f-8e1c-470b-b324-557d7580b1e4" />  

Исправим конфигурацию.  
<img width="572" height="632" alt="image" src="https://github.com/user-attachments/assets/955a5101-2938-4db1-82bb-889dfe54c4d3" />  

Под запустился.  
<img width="979" height="442" alt="image" src="https://github.com/user-attachments/assets/eda7b7c0-7a7d-4c40-9b39-401f2818e0c3" />

## Часть 7. Сравнение трех состояний: NotReady, CrashLoopBackOff, Pending
Краткое описание каждого состояния.

NotReady — pod работает, но не готов принимать трафик (readiness probe не прошла). Проблема в самом приложении или настройках пробы. Быстрая диагностика: kubectl get pods показывает 0/1 в колонке READY.

CrashLoopBackOff — контейнер постоянно падает и перезапускается. Причина: ошибка в приложении, OOMKilled, Неправильная настройка лимитов. Быстрая диагностика: kubectl get pods показывает RESTARTS растущим, статус CrashLoopBackOff.

Pending — pod не может быть размещён на ноде. Причина: не хватает CPU/памяти, завышены requests. Быстрая диагностика: kubectl describe pod → Events с FailedScheduling.
























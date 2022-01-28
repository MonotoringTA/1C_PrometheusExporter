# 1C_PrometheusExporter
Полнофункциональное расширение 1C позволяющее выгружать метрики из 1С 8.3 в Prometheus Pushgetway. На текущий момент реализована выгрузка метрик с типом histogram для расчета APDEX в Grafana. Работает с 1С версии 8.3.17 и выше, так как используется константа для хранения адреса сервера pushgateway (14 точно не работает).

Описание базовой логики:
Для логирования скорости выполнения операций в БСП используется стандартная подсистема «Оценка производительности». Запись происходит в разрезе ключевых операций. Все данные записываются в типовой регистр сведений «ЗамерыВремени». После записи в регистр «ЗамерыВремени» в расширении происходит формирование регистра «APDEXMetrics» для выгрузки из него метрик в нужном для Prometheus формате.

По условиям задачи, данные должны выгружаться в Prometheus Pushgetway 1 раз в минуту. Поэтому для того чтобы обеспечить достаточную скорость формирования метрик для нескольких десятков тысяч ключевых операций в регистре всегда хранится только текущий срез данных разбитых по бакетам (временные интервала в которые попадают записи). Все записи считаются нарастающим итогом в разрезе ключевой операции.

В сборке присутствуют 
1.	Расширение для 1С «PrometheusExporter.cfe»
2.	Prometheus pushgetway
3.	Prometheus
4.	Grafana
5.	Обработка «Тест формирования регистра APDEX» - позволяет делать записи штатным механизмом логирования времени выполнения ключевых операций, а также для чистки регистра «APDEXMetrics».git
6. pushgetway.sh - пример выгрузки метрик разных типов через curl (полезно, когда хочешь понять механику работы)
7. Grafana\1C APDEX.json - дашборд Grafana для расчета APDEX

Краткая инструкция:
1.	Загрузите себе на машину данную сборку при помощи git-а
2.	Установите типовую конфигурацию 1С БСП
3.	Подключите к БСП расширение 1С «PrometheusExporter.cfe»
4.	Установите Docker desktop
5.	Перейдите в файл «docker-compose.yaml» запустите terminal и выполните команду docker-compose up -d
6.	Проверьте что что по адресу http://localhost:9091  у вас поднялся Prometheus pushgetway 
7.	Проверьте что что по адресу http://localhost:9090  у вас поднялся Prometheus
8.	Проверьте что что по адресу http://localhost:3000  у вас поднялась Grafana (логин: admin, пароль: admin)
9.	Войдите в 1С и в константу «Адрес сервера pushgateway» укажите адрес http://localhost:9091/
10.	Отройте регистр сведений «APDEXMetrics» и проверьте что там есть записи
11.	Откройте обработку «Prometheus - выгрузить APDEX в Pushgetway» и нажмите кнопку «Отправить в Pushgetway».
12.	Откройте http://localhost:9091 и проверьте, что метрики записались. Также можно открыть метрики в том виде, в котором их забирают Prometheus: http://localhost:9091/metrics
13.	При помощи обработки «Тест формирования регистра APDEX» можно экспериментировать.
  - Удалить все записи из регистра «APDEXMetrics»
  - Запишите данные с нужным вам временем (данные с клиента пишутся асинхронно, поэтому для тестирования лучше использовать запись на сервере).
  - Очистите Prometheus pushgetway (docker-compose down -> docker-compose up -d)
  - Отправьте метрики снова при помощи обработки «Prometeus - выгрузить APDEX в Pushgetway»
  - Для внесения нескольких данных, просто закройте и снова откройте БСП (сработает типовой алгоритм логирования ключевых операций при входе)
14.	Если не работает выгрузка в Prometheus pushgetway есть файл pushgetway.sh. Используя файл можно отправлять метрики вручную. Отправка просиходит через Bash (если у вас устновлен Git, то Bash у вас есть). Просто откройте терминал выберите Git Bash, напишите «./ pushgetway.sh» и Prometheus pushgetway вы должны будите увидите 4 метрики разных типов.
15.	Открывайте Grafana и делайте свои борды или попробуйте наш 1C Grafana\APDEX.json.

Всем удачи!


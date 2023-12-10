# Домашнее задание №4
**Задача:**

1. Отключение узла: Планово остановить один из узлов кластера, чтобы проверить процедуру
переключения ролей (failover). - Анализировать время, необходимое для восстановления и как
система выбирает новый Master узел (и есть ли вообще там стратегия выбора?).
2. Имитация частичной потери сети: Использовать инструменты для имитации потери пакетов
или разрыва TCP-соединений между узлами. Цель — проверить, насколько хорошо система
справляется с временной недоступностью узлов и как быстро восстанавливается репликация.
3. Высокая нагрузка на CPU или I/O: Запустить процессы, которые создают высокую нагрузку на CPU или дисковую подсистему одного из узлов кластера, чтобы проверить, как это влияет на
производительность кластера в целом и на работу Patroni.
4. Тестирование систем мониторинга и оповещения: С помощью chaos engineering можно также
проверить, насколько эффективны системы мониторинга и оповещения. Например, можно
искусственно вызвать отказ, который должен быть зарегистрирован системой мониторинга, и
убедиться, что оповещения доставляются вовремя ?

**Если сделали все предыдущие:**
1. ”Split-brain": Одновременно изолировать несколько узлов от сети и дать им возможность
объявить себя новыми мастер-узлами. Проверить, успеет ли Patroni достичь
консенсуса и избежать ситуации "split-brain".
2. Долгосрочная изоляция: Оставить узел изолированным от кластера на длительное время, затем восстановить соединение и наблюдать за процессом синхронизации и
восстановления реплики.
3. Сбои сервисов зависимостей: Изучить поведение кластера Patroni при сбоях в сопутствующих сервисах, например, etcd (которые используются для хранения состояния кластера),
путем имитации его недоступности или некорректной работы.

**Формат сдачи ДЗ:**
Ссылка на репозиторий, где размещен md файл с описанием экспериментов и результатами

1. Описание эксперимента: Подробные шаги, которые были предприняты для
имитации условий эксперимента. - Инструменты и методы, применяемые в
процессе.
2. Ожидаемые результаты: Описание ожидаемого поведения системы в ответ
на условия эксперимента.
3. Реальные результаты: Что произошло на самом деле в ходе эксперимента.
Логи, метрики и выводы систем мониторинга и логирования.
4. Анализ результатов: Подробное сравнение ожидаемых и реальных
результатов. - Обсуждение возможных причин отклонений.

Тестовая система:



09.12.23
12:01 Выключение ведущего узла
12:05 Аппаратное выключение ведущего узла


sudo blade create network corrupt --percent 50 --destination-ip 10.0.10.4 --interface ens160 --timeout 300
sudo blade create network corrupt --percent 50 --destination-ip 10.0.10.3 --interface ens160 --timeout 300

sudo blade create network drop --destination-ip 10.0.10.4  --timeout 300
sudo blade create network drop --destination-ip 10.0.10.3  --timeout 300

10/12/24
11:33
отключаем доступ к etcd
sudo blade create network drop --destination-ip 10.0.10.5,10.0.10.6,10.0.10.7  --timeout 300
Screenshot 2023-12-10 at 19.46.29

11:59
отключаем доступ к etcd частично
и выключаем postgresql мастер
sudo blade create network drop --destination-ip 10.0.10.5,10.0.10.6  --timeout 300

12:11
отключаем один узел etcd
и выключаем postgresql/patroni мастер
sudo blade create network drop --destination-ip 10.0.10.5,10.0.10.6  --timeout 300

12:22
изолировать etcd leader db1 от etcd
sudo blade create network drop --destination-ip 10.0.10.5,10.0.10.6,10.0.10.7  --timeout 300

изолировать postgresql/patroni мастер srv2
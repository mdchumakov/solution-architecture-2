Что нужно сделать: провести анализ текущей архитектуры, выявить проблемы и риски, разработать целевое архитектурное видение и задокументировать ключевые решения посредством ADR.

Конкретные задачи
1. As-is анализ (диагностика текущей архитектуры)
Изучите диаграмму C4 Container текущей архитектуры AdScale — компоненты монолита, внешние системы (DSP-партнёры и платёжные шлюзы), базу данных и направления потоков. 
Проведите анализ, чтобы выявить узкие места и точки отказа, определить архитектурные проблемы, препятствующие масштабированию, и оценить влияние текущей архитектуры на время отклика.  
Диаграмма текущего решения:

```plantuml
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

LAYOUT_WITH_LEGEND()
LAYOUT_LEFT_RIGHT()

title C4 Container Diagram — AdScale (AS-IS)

' ====================== Акторы ======================
Person(advertiser, "Рекламодатель", "Управляет кампаниями, бюджетами и ставками через личный кабинет")
Person_Ext(endUser, "Пользователь сайта/приложения", "Просматривает рекламу, кликает по баннерам")

System_Ext(dspPartner, "DSP-партнёр", "Отправляет bid-requests, получает bid-responses по OpenRTB")
System_Ext(paymentGateway, "Платёжный шлюз", "Обработка пополнения рекламных бюджетов")

' ====================== Монолит ======================
System_Boundary(monolith, "AdScale Monolith (C++ / Go / Python / Node.js)") {

    Container(adServer, "Ad Server", "C++, Go", "Принимает HTTP-запросы, определяет пользователя, подбирает кандидатов в рекламу") <<Bottleneck>>

    Container(auctionEngine, "Auction Engine", "C++", "Взвешивание ставок, бизнес-правила, расчёт победителя аукциона") <<Critical Path>>

    Container(deliveryService, "Delivery Service", "Node.js", "Формирует HTML/JS-разметку баннера и HTTP-ответ")

    Container(statsService, "Statistics Service", "Python", "Запись кликов и показов") <<Bottleneck>>

    Container(financialService, "Financial Service", "Go", "Списание средств, выставление счетов, управление балансами")

    Container(analyticsService, "Analytics Service", "Python", "Аналитические SQL-запросы по production-БД") <<Bottleneck>>

    Container(adCabinet, "Advertiser Dashboard", "Node.js + JavaScript", "Web UI: управление кампаниями, отчётность")
}

' ====================== База данных ======================
ContainerDb(postgres, "PostgreSQL", "PostgreSQL (одна инстанция)", "Единая БД: кампании, ставки, события, финансы, аналитика") <<Single Point of Failure>>

' ====================== Связи ======================

' Внешние связи
Rel_U(dspPartner, adServer, "Bid-requests / responses", "HTTPS / OpenRTB")
Rel_U(endUser, deliveryService, "Просмотр рекламы и клики", "HTTPS")
Rel(advertiser, adCabinet, "Управляет кампаниями и бюджетами", "HTTPS")
Rel(adCabinet, paymentGateway, "Пополнение бюджета", "HTTPS / REST")

' Внутренние связи внутри монолита
Rel(adServer, auctionEngine, "Передаёт кандидатов", "Internal Call")
Rel(auctionEngine, deliveryService, "Передаёт победителя аукциона", "Internal Call")
Rel(deliveryService, statsService, "Регистрирует показы/клики", "Internal Call")
Rel(adCabinet, financialService, "Управление финансами", "Internal Call")
Rel(adCabinet, analyticsService, "Запросы аналитики", "Internal Call")

' Связи с БД
Rel(auctionEngine, postgres, "Читает ставки и кампании", "JDBC / Sync")
Rel(statsService, postgres, "Прямая запись событий", "JDBC / Direct Write")
Rel(financialService, postgres, "Транзакции и балансы", "JDBC")
Rel(analyticsService, postgres, "Тяжёлые аналитические запросы", "JDBC")
Rel(adCabinet, postgres, "CRUD-операции", "JDBC")
Rel(adServer, postgres, "Чтение данных для таргетинга", "JDBC")

' Принудительное расположение (layout hints)
Lay_D(adServer, auctionEngine)
Lay_D(auctionEngine, deliveryService)
Lay_D(deliveryService, statsService)
Lay_R(statsService, financialService)
Lay_R(financialService, analyticsService)
Lay_D(adCabinet, postgres)

' ====================== Аннотации проблемных мест ======================

note right of auctionEngine
  **⚠ Критический bottleneck**
  Синхронные вызовы к БД
  в hot path (SLA ≤ 100 мс)
end note

note right of postgres
  **⚠ Single Point of Failure**
  Одна инстанция — нет HA
  Нет read-replicas
  OLTP + аналитика конкурируют
end note

note bottom of statsService
  **⚠ write bottleneck**
  Прямая запись без буферизации / очередей
  Конкуренция за ресурсы БД
end note

note bottom of analyticsService
  **⚠ Блокировки и рост latency**
  Heavy queries напрямую на production-БД
end note

' ====================== Легенда с метриками ======================
legend right
  <#FF0000> **15 000 RPS (пик)** 
  <#FF7700> **P95 latency: 110–120 мс** (требование ≤ 100 мс)
  <#FF7700> Периодические деградации в часы пик
  <#FF0000> Нет запаса по производительности
  <#888888> Деплой: вручную, без контейнеризации
  <#888888> Кеширование: отсутствует
  <#888888> Асинхронные очереди: отсутствуют
endlegend
@enduml
```

2. Определение архитектурных драйверов
Сформулируйте функциональные требования к системе после интеграции, атрибуты качества (с приоритизацией), технические и бизнес-ограничения.
3. Выбор архитектурного стиля
Обоснуйте выбор архитектурного стиля для эволюции системы. В частности, почему не оставить монолит и почему не стоит сразу переписывать на микросервисы? Какой подход будет оптимальнее: Strangler Fig Pattern, модульный монолит или постепенная декомпозиция?
4. Целевая архитектура
Создайте диаграмму C4 Container для целевой архитектуры через год и отразите на ней выделенные микросервисы с обоснованием границ, взаимодействие между сервисами, инфраструктурные компоненты (брокеры, кеш, API Gateway) и базы данных для каждого сервиса. 
5. Architecture Decision Records (ADR)
Оформите как минимум одно архитектурное решение в формате ADR на выбор:
ADR-001. Стратегия эволюции архитектуры (от монолита к микросервисам).
ADR-002. Выделение сервиса ставок в качестве первого приоритета.
ADR-003. Выбор технологии для потоковой обработки событий.
В результате у вас должно получиться: 
файл README.md с кратким описанием;
AS-IS.md с анализом текущей архитектуры;
Drivers.md — архитектурные драйверы;
TO-BE.md с описанием целевой архитектуры;
Diagrams/ — диаграммы C4 (TO-BE) в формате PlantUML или Draw.io;
adr/ — документы ADR.
Загрузите получившиеся артефакты в директорию Task1 вашего репозитория.
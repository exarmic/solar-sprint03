@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml
title As-Is: кассовая зона магазина VitaStore (C4 Container)\nАртефакт 1, уточнённый по артефактам 2–4

Person(cashier, "Кассир", "Сотрудник магазина")
Person(engineer, "Выездной ИТ-инженер", "Настраивает сервер магазина, снимает дамп товаров старой кассы")
Person(merch, "Товаровед (HQ)", "Вручную сводит номенклатуру в Excel")

Boundary(hq, "Центральный офис (on-premise)") {
  System_Ext(erp, "SAP ERP", "Учёт товаров, кадров, финансов. Эталонная номенклатура, цены, лояльность")
}

System_Boundary(store, "Кассовая система магазина") {
  Container(pos, "POS App (толстый клиент)", "Windows, C#", "Монолитное десктопное приложение. Содержит бизнес-логику и драйверы устройств")
  Container(backoffice, "Back-office", "Десктопное ПО", "Товарный учёт магазина: приём товара, ценники")
  ContainerDb(db, "Локальная база данных", "MS SQL Server / PostgreSQL", "На сервере в подсобке. Хранит чеки и справочники магазина")
  Container(exchange, "Файловый обмен", "FTP, XML", "На сервере в подсобке. Принимает ночной пакет и загружает его в базу")
}

System_Ext(printer, "Фискальный регистратор", "Аппарат ККТ (COM-порт), разные модели")
System_Ext(terminal, "Банковский терминал", "Разнородный парк: Ingenico, Verifone, терминалы Сбера и др.")

Rel(cashier, pos, "Пробивает чеки", "UI")
Rel_R(pos, db, "Читает цены, пишет чеки", "SQL")
Rel(backoffice, db, "Приём товара, печать ценников", "SQL")
Rel(erp, exchange, "Пакет обновлённого прайса в 02:00", "FTP, XML")
Rel(exchange, db, "Загружает цены и номенклатуру", "SQL", "Через PS-скрипт (Допущение)")
Rel(db, exchange, "Дамп продаж и остатков (допущение)", "SQL", "Через PS-скрипт (Допущение)")
Rel(exchange, erp, "Выгрузка продаж и остатков (допущение)", "FTP, файл")
Rel(pos, erp, "Карта лояльности: проверка и баллы. При обрыве связи — тайм-аут (INC-8402)", "онлайн-запрос")
Rel(pos, printer, "Прямое управление печатью", "COM-port driver")
Rel(pos, terminal, "Инициация оплаты", "USB driver, протокол банка")
Rel(engineer, db, "Настраивает сервер на месте", "выезд")
Rel(engineer, merch, "Дамп товаров (.csv / .xls)", "e-mail")
Rel(merch, erp, "Загружает сведённую таблицу", "вручную")

note right of pos
  POS App жёстко привязан к моделям
  оборудования через встроенные драйверы.
  Новый терминал = новый драйвер
  и пересборка кассы, 3 недели (INC-8415).
end note

note right of db
  Единая точка отказа: при поломке
  сервера магазин стоит 4–6 часов (INC-8433).
end note
@enduml
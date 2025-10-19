# **Программное средство реализации онлайн-сервиса «Перепланировка, переустройство и перевод помещения»**

Краткое описание проекта, его цели и основные возможности

Ссылки на репозитории сервера и клиента

---

## **Содержание**

1. [Архитектура](#Архитектура)
	1. [C4-модель](#C4-модель)
	2. [Схема данных](#Схема_данных)
2. [Функциональные возможности](#Функциональные_возможности)
	1. [Диаграмма вариантов использования(#Диаграмма_вариантов_использования)]
	2. [User-flow диаграммы](#User-flow_диаграммы)
 	3. [Примеры экранов UI](#Примеры экранов UI)
3. [Детали реализации](#Детали_реализации)
	1. [UML-диаграммы](#UML-диаграммы)
	2. [Спецификация API](#Спецификация_API)
	3. [Безопасность](#Безопасность)
	4. [Оценка качества кода](#Оценка_качества_кода)
4. [Тестирование](#Тестирование)
	1. [Unit-тесты](#Unit-тесты)
	2. [Интеграционные тесты](#Интеграционные_тесты)
5. [Установка и  запуск](#installation)
	1. [Манифесты для сборки docker образов](#Манифесты_для_сборки_docker_образов)
	2. [Манифесты для развертывания k8s кластера](#Манифесты_для_развертывания_k8s_кластера)
6. [Лицензия](#Лицензия)
7. [Контакты](#Контакты)

---
## **Архитектура**

### C4-модель

Контейнерный уровень архитектуры программного средства реализации онлайн-сервиса «Перепланировка, переустройство и перевод помещения» в нотации С4 представлен на рисунке 1.1.

 <img width="856" height="535" alt="image" src="https://github.com/user-attachments/assets/e64d84cb-58d6-486a-9a15-1268cad09761" />

Рисунок 1.1 – Контейнерный уровень архитектуры программного средства в нотации С4
В центре системы находится одностраничное приложение, разработанное с использованием JavaScript и Angular, которое предоставляет интерфейс для сотрудников ЖЭС, исполкома, БТИ и для клиентов, желающих произвести перепланировку, переустройство или перевода собственного объекта недвижимости.
Для обработки запросов от веб-приложений используется API-приложение, разработанное на Java с применением Spring MVC. Это приложение предоставляет доступ к функциональности системы управления базой клиентов и контактов через REST API и взаимодействует с базой данных и компонентами, получая данные от контроллеров. 
База данных, реализованная на основе MySQL, служит хранилищем данных по клиентам, их недвижимости, по документам изменений статуса объекта недвижимости, истории ответов от ЖЭС и БТИ на заявления по изменению статуса. MySQL отвечает за чтение и запись данных для контроллеров и API-приложения. 
В системе также присутствует компонент электронной почты Google Space. Этот компонент отвечает за отправку писем на почту клиентам об актуальных коммерческих предложениях и маркетинговых активностях, взаимодействуя с контроллерами для инициирования отправок. 
Компонентный уровень в нотации С4 архитектуры программного средства реализации онлайн-сервиса «Перепланировка, переустройство и перевод помещения» представлен на рисунке 1.2.

<img width="929" height="615" alt="image" src="https://github.com/user-attachments/assets/36213859-f1f3-47fb-b355-984455a6e54f" />

Рисунок 1.2 – Компонентный уровень архитектуры программного средства в нотации С4
На рисунке можно увидеть, что контейнер с API-приложением был разобран на компоненты – контроллеры. Одностраничное приложение взаимодействует с каждым контроллером. На изображении 1.2 представлена архитектура программного средства, реализованного с использованием Angular и JavaScript на фронтенде, а также Spring MVC и MySQL на бэкенде. В центре внимания – взаимодействие между пользовательским интерфейсом, контроллерами серверной части и базой данных. Фронтенд отправляет HTTP-запросы к различным контроллерам, каждый из которых отвечает за свою область: обработку заявок, управление клиентами, работу с объектами, безопасность, профили пользователей и отправку уведомлений. Все контроллеры реализованы на Spring MVC и взаимодействуют с базой данных MySQL, выполняя операции чтения и записи. Кроме того, Push Controller передаёт данные в отдельную систему отправки сообщений, которая занимается доставкой уведомлений. Поток данных организован таким образом, что пользователь инициирует действия через веб-интерфейс, запросы поступают на соответствующий контроллер, обрабатываются с участием базы данных, и результат возвращается обратно на фронт. Такая архитектура обеспечивает модульность, масштабируемость и чёткое разделение ответственности между компонентами системы.

<img width="682" height="894" alt="image" src="https://github.com/user-attachments/assets/be833967-bac4-418f-8e01-4db92a428a4e" />

Рисунок 3.1 – Диаграмма классов программного средства

Пользователь инициирует запрос через RenovationServiceAdapter. Реализация (RenovationServiceAdapterImpl) формирует SubmitRenovationRequest, включающий заявителя, помещение и план. Запрос сериализуется в JSON и передаётся через RenovationSystemConnection. Система обрабатывает запрос и возвращает JSON-ответ. Ответ десериализуется в SubmitRenovationResponse, содержащий статус и ID заявки.


### Схема данных

![photo_2025-10-04_16-15-54](https://github.com/user-attachments/assets/20c7a5ba-2ad8-4d98-aa19-2da7b4b1ae07)

Скрипт генерации БД:
DO $$ BEGIN
    IF NOT EXISTS (SELECT 1 FROM pg_type WHERE typname = 'operation_type_enum') THEN
        CREATE TYPE operation_type_enum AS ENUM (
            'REDEVELOPMENT',    -- перепланировка
            'CONVERSION',       -- перевод (жилое <-> нежилое)
            'RECONSTRUCTION'    -- переустройство/реконструкция
        );
    END IF;
END$$;

DO $$ BEGIN
    IF NOT EXISTS (SELECT 1 FROM pg_type WHERE typname = 'application_status_enum') THEN
        CREATE TYPE application_status_enum AS ENUM (
            'NEW',
            'IN_REVIEW',
            'AWAITING_INSPECTION',
            'INSPECTION_DONE',
            'APPROVED',
            'REJECTED',
            'CLOSED'
        );
    END IF;
END$$;

DO $$ BEGIN
    IF NOT EXISTS (SELECT 1 FROM pg_type WHERE typname = 'property_type_enum') THEN
        CREATE TYPE property_type_enum AS ENUM (
            'RESIDENTIAL',
            'NON_RESIDENTIAL'
        );
    END IF;
END$$;

DO $$ BEGIN
    IF NOT EXISTS (SELECT 1 FROM pg_type WHERE typname = 'object_type_enum') THEN
        CREATE TYPE object_type_enum AS ENUM (
            'APARTMENT',
            'HOUSE',
            'OFFICE',
            'COMMERCIAL',
            'OTHER'
        );
    END IF;
END$$;

DO $$ BEGIN
    IF NOT EXISTS (SELECT 1 FROM pg_type WHERE typname = 'attachment_category_enum') THEN
        CREATE TYPE attachment_category_enum AS ENUM (
            'BTI_PASSPORT',
            'PROJECT',
            'OWNER_CONSENT',
            'PHOTO',
            'OTHER'
        );
    END IF;
END$$;


CREATE TABLE IF NOT EXISTS roles (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE
);


CREATE TABLE IF NOT EXISTS users (
    id BIGSERIAL PRIMARY KEY,
    passport_identifier VARCHAR(128) UNIQUE,
    phone_number VARCHAR(30),
    full_name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE,
    photo_url TEXT,
    enabled BOOLEAN NOT NULL DEFAULT true,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE OR REPLACE FUNCTION trg_set_timestamp()
RETURNS TRIGGER AS $$
BEGIN
   NEW.updated_at := now();
   RETURN NEW;
END;
$$ LANGUAGE plpgsql;

DROP TRIGGER IF EXISTS users_updated_at_trg ON users;
CREATE TRIGGER users_updated_at_trg
BEFORE UPDATE ON users
FOR EACH ROW
EXECUTE PROCEDURE trg_set_timestamp();


CREATE TABLE IF NOT EXISTS users_roles (
    user_id BIGINT NOT NULL,
    role_id BIGINT NOT NULL,
    PRIMARY KEY (user_id, role_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE
);

CREATE INDEX IF NOT EXISTS idx_users_roles_user ON users_roles(user_id);
CREATE INDEX IF NOT EXISTS idx_users_roles_role ON users_roles(role_id);


CREATE TABLE IF NOT EXISTS properties (
    id BIGSERIAL PRIMARY KEY,
    owner_id BIGINT NOT NULL,
    property_type property_type_enum NOT NULL DEFAULT 'RESIDENTIAL',
    object_type object_type_enum NOT NULL DEFAULT 'APARTMENT',
    full_address TEXT NOT NULL,
    cadastral_number VARCHAR(128),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    FOREIGN KEY (owner_id) REFERENCES users(id) ON DELETE RESTRICT
);

DROP TRIGGER IF EXISTS properties_updated_at_trg ON properties;
CREATE TRIGGER properties_updated_at_trg
BEFORE UPDATE ON properties
FOR EACH ROW
EXECUTE PROCEDURE trg_set_timestamp();

CREATE INDEX IF NOT EXISTS idx_properties_owner ON properties(owner_id);
CREATE INDEX IF NOT EXISTS idx_properties_address ON properties USING gin (to_tsvector('russian', full_address));


CREATE TABLE IF NOT EXISTS applications (
    id BIGSERIAL PRIMARY KEY,
    applicant_id BIGINT NOT NULL,       -- заявитель (user)
    property_id BIGINT NOT NULL,        -- объект недвижимости
    operation_type operation_type_enum NOT NULL,
    status application_status_enum NOT NULL DEFAULT 'NEW',
    rejection_reason TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    FOREIGN KEY (applicant_id) REFERENCES users(id) ON DELETE RESTRICT,
    FOREIGN KEY (property_id) REFERENCES properties(id) ON DELETE RESTRICT
);

DROP TRIGGER IF EXISTS applications_updated_at_trg ON applications;
CREATE TRIGGER applications_updated_at_trg
BEFORE UPDATE ON applications
FOR EACH ROW
EXECUTE PROCEDURE trg_set_timestamp();

CREATE INDEX IF NOT EXISTS idx_applications_applicant ON applications(applicant_id);
CREATE INDEX IF NOT EXISTS idx_applications_property ON applications(property_id);
CREATE INDEX IF NOT EXISTS idx_applications_status ON applications(status);


CREATE TABLE IF NOT EXISTS application_history (
    id BIGSERIAL PRIMARY KEY,
    application_id BIGINT NOT NULL,
    status application_status_enum NOT NULL,
    description TEXT,
    actor_name VARCHAR(255),
    event_timestamp TIMESTAMPTZ NOT NULL DEFAULT now(),
    FOREIGN KEY (application_id) REFERENCES applications(id) ON DELETE CASCADE
);

CREATE INDEX IF NOT EXISTS idx_app_history_app ON application_history(application_id);
CREATE INDEX IF NOT EXISTS idx_app_history_time ON application_history(event_timestamp);


CREATE TABLE IF NOT EXISTS acts (
    id BIGSERIAL PRIMARY KEY,
    application_id BIGINT NOT NULL,
    inspection_date_1 DATE,
    inspection_date_2 DATE,
    conclusion TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    FOREIGN KEY (application_id) REFERENCES applications(id) ON DELETE CASCADE
);

CREATE INDEX IF NOT EXISTS idx_acts_application ON acts(application_id);


CREATE TABLE IF NOT EXISTS attachments (
    id BIGSERIAL PRIMARY KEY,
    application_id BIGINT NOT NULL,
    category attachment_category_enum NOT NULL DEFAULT 'OTHER',
    file_path TEXT NOT NULL,
    original_file_name VARCHAR(1000),
    mime_type VARCHAR(255),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    FOREIGN KEY (application_id) REFERENCES applications(id) ON DELETE CASCADE
);

CREATE INDEX IF NOT EXISTS idx_attachments_application ON attachments(application_id);


CREATE TABLE IF NOT EXISTS flyway_schema_history (
    installed_rank INT PRIMARY KEY,
    version VARCHAR(50),
    description VARCHAR(200),
    type VARCHAR(20),
    script VARCHAR(1000),
    checksum INT,
    installed_by VARCHAR(100),
    installed_on TIMESTAMPTZ NOT NULL DEFAULT now(),
    execution_time INT,
    success BOOLEAN
);

ALTER TABLE applications
    ADD CONSTRAINT applications_rejection_reason_check
    CHECK (
        (status = 'REJECTED' AND rejection_reason IS NOT NULL)
        OR (status <> 'REJECTED')
    );

CREATE INDEX IF NOT EXISTS idx_applications_created_at ON applications(created_at);

INSERT INTO roles (name)
SELECT v
FROM (VALUES ('CLIENT'), ('ZHEK'), ('BTI'), ('ADMIN')) AS t(v)
ON CONFLICT (name) DO NOTHING;




---

## **Функциональные возможности**

### Диаграмма вариантов использования

Диаграмма вариантов использования и ее описание

### User-flow диаграммы

User flow для клиента
![User flow для клиента](https://github.com/user-attachments/assets/8de5f556-e9b7-42c7-8bc7-6722c4a16011)


User flow для ЖЭК
![User flow для ЖЭК](https://github.com/user-attachments/assets/80a2fc27-0a14-4906-a462-b7c639bc2a09)


User flow для БТИ
![User flow для БТИ](https://github.com/user-attachments/assets/80a7cf21-bab3-42f8-ace2-eedc19610e9d)

### Примеры экранов UI

Ниже на рисунках 1-7 представлены страницы для роли «Клиент» (пользователь, что подает заявления на рассмотрение в ЖЭК и БТИ).
Страница «Главная» представлена на рисунке 1. 

<img width="982" height="467" alt="image" src="https://github.com/user-attachments/assets/02377383-c5ef-434e-be46-b445288886cd" />
Рисунок 1 – Страница «Главная»

Страница «Заявления» представлена на рисунке 2. 

<img width="983" height="446" alt="image" src="https://github.com/user-attachments/assets/4007e68d-3a3c-4d53-a5e0-820bbbb2893e" />
Рисунок 2 – Страница «Заявления»

Страница «Детали заявления» представлена на рисунке 3. 

<img width="982" height="473" alt="image" src="https://github.com/user-attachments/assets/0760588c-303e-4749-bc63-18b3b3060fcb" />
Рисунок 3 – Страница «Детали заявления»

Страница «История уведомлений» представлена на рисунке 4. 

<img width="982" height="394" alt="image" src="https://github.com/user-attachments/assets/1fb4f14c-5200-45c3-a9ed-4add1d3f2dbd" />
Рисунок 4 – Страница «История уведомлений»

Страница «Личные данные» представлена на рисунке 5. 

<img width="982" height="467" alt="image" src="https://github.com/user-attachments/assets/33e62af4-70a5-4146-b722-0e79faefd694" />
Рисунок 5 – Страница «Личные данные»

Страница «Объекты недвижимости» представлена на рисунке 6. 

<img width="983" height="310" alt="image" src="https://github.com/user-attachments/assets/37a2b4da-426f-4251-87d6-9d5e92c65cc5" />
Рисунок 6 – Страница «Объекты недвижимости»

Страница «Создание заявления» представлена на рисунке 7. 

<img width="982" height="466" alt="image" src="https://github.com/user-attachments/assets/5c7912e7-2c18-48e9-a84a-5ebd1dcb1423" />
Рисунок 7 – Страница «Создание заявления»

Ниже на рисунках 8-10 представлены страницы для роли «ЖЭК».
Страница «Заявления» представлена на рисунке 8. 

<img width="982" height="333" alt="image" src="https://github.com/user-attachments/assets/18c61689-944e-4fb7-8385-00f3ef21a74c" />
Рисунок 8 – Страница «Заявления»

Страница с модальным окном «Согласование заявления» (после согласования ЖЭК, заявления направится на рассмотрение в БТИ) представлена на рисунке 9. 

<img width="978" height="463" alt="image" src="https://github.com/user-attachments/assets/3812ae0d-f572-4042-8fb0-aa83cd96f2e5" />
Рисунок 9 – Страница с модальным окном «Согласование заявления»

Страница «Согласованные заявления для БТИ» представлена на рисунке 10. 

<img width="981" height="368" alt="image" src="https://github.com/user-attachments/assets/ccac3fac-e7e6-45b9-adfc-1c82cdde2302" />
Рисунок 10 – Страница «Согласованные заявления для БТИ»

Ниже на рисунках 11-13 представлены страницы для роли «БТИ».
Страница «Обжалованные заявления» представлена на рисунке 11. 

<img width="982" height="452" alt="image" src="https://github.com/user-attachments/assets/f114038c-9acd-452c-a8d8-bcdc8dffcfe3" />
Рисунок 11 – Страница «Обжалованные заявления»

Страница с модальным окном «Согласование заявления БТИ» представлена на рисунке 12. 

<img width="978" height="462" alt="image" src="https://github.com/user-attachments/assets/2bfca2b4-feb5-4009-b10d-da721687bac7" />
Рисунок 12 – Страница с модальным окном «Согласование заявления БТИ»

Страница «Акты» представлена на рисунке 13. 

<img width="982" height="467" alt="image" src="https://github.com/user-attachments/assets/28f89b5f-f29c-4921-9a67-ecf4a9f38996" />
Рисунок 13 – Страница «Акты»


---

## **Детали реализации**

### UML-диаграммы

Представить все UML-диаграммы , которые позволят более точно понять структуру и детали реализации ПС

### Спецификация API

Представить описание реализованных функциональных возможностей ПС с использованием Open API (можно представить либо полный файл спецификации, либо ссылку на него)

### Безопасность

Описать подходы, использованные для обеспечения безопасности, включая описание процессов аутентификации и авторизации с примерами кода из репозитория сервера

### Оценка качества кода

Используя показатели качества и метрики кода, оценить его качество

---

## **Тестирование**

### Unit-тесты

Представить код тестов для пяти методов и его пояснение

### Интеграционные тесты

Представить код тестов и его пояснение

---

## **Установка и  запуск**

### Манифесты для сборки docker образов

Представить весь код манифестов или ссылки на файлы с ними (при необходимости снабдить комментариями)

### Манифесты для развертывания k8s кластера

Представить весь код манифестов или ссылки на файлы с ними (при необходимости снабдить комментариями)

---

## **Лицензия**

Этот проект лицензирован по лицензии MIT - подробности представлены в файле [[License.md|LICENSE.md]]

---

## **Контакты**

Автор: email

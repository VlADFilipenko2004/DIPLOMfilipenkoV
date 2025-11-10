# **Программное средство реализации онлайн-сервиса «Перепланировка, переустройство и перевод помещения»**

Краткое описание проекта, его цели и основные возможности

Ссылки на репозитории сервера и клиента

---

## **Содержание**

1. [Архитектура](#Архитектура)
	1. [C4-модель](#C4-модель)
	2. [Схема данных](#Схема_данных)
2. [Функциональные возможности](#Функциональные_возможности)
	1. [Диаграмма вариантов использования](#Диаграмма_вариантов_использования)
	2. [User-flow диаграммы](#User-flow_диаграммы)
 	3. [Примеры экранов UI](#Примеры_экранов_UI)
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

 <img width="983" height="430" alt="image" src="https://github.com/user-attachments/assets/d282838d-5912-4042-bc0b-a69c6d7c534f" />

Рисунок 1.1 – Диаграмма деятельности процесса смены статуса заявления «Клиент – Сотрудник ЖЭК»

 <img width="942" height="885" alt="image" src="https://github.com/user-attachments/assets/ea99f5be-9724-4fce-8829-929106df3d83" />

Рисунок 1.2 – Диаграмма состояний объекта Статус клиента

<img width="1556" height="1162" alt="Диаграмма последовательности процесса изменения статуса заявления (Клиент - Сотрудник ЖЭК) Варик 2" src="https://github.com/user-attachments/assets/49960cd9-71e2-4d28-b275-8fef1647ae3f" />

Рисунок 1.3 – Диаграмма последовательности процесса изменения статуса «Клиент – Сотрудник ЖЭК»

 <img width="713" height="738" alt="image" src="https://github.com/user-attachments/assets/81456f31-6530-4bca-b141-67c7f3114998" />

Рисунок 1.4 – Диаграмма классов программного средства

 <img width="614" height="414" alt="image" src="https://github.com/user-attachments/assets/99df4979-e968-4b11-8bca-ad5411835e41" />

Рисунок 1.5 – Диаграмма развертывания системы

### Спецификация API

Представить описание реализованных функциональных возможностей ПС с использованием Open API (можно представить либо полный файл спецификации, либо ссылку на него)

### Безопасность

При разработке системы аутентификации и авторизации одной из ключевых задач является создание надёжного и гибкого механизма управления доступом, который эффективно защищает чувствительные данные и функции. Специфика предметной области требует, чтобы система могла быстро и точно идентифицировать пользователей, проверяя их полномочия и обеспечивая доступ только к разрешённым ресурсам.
Проектирование системы управления доступом требует не только тщательной проверки идентификационных данных, но и учёта множества технических и функциональных требований. Система должна поддерживать высокую скорость обработки запросов, обеспечивать масштабируемость и безопасность данных, а также поддерживать различные методы аутентификации и уровни доступа.
Для реализации механизма авторизации и аутентификации были использованы Spring Security, входящий в состав Spring Framework, и JWT (JSON Web Token).

@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {
    private final JwtAuthenticationFilter jwtAuthFilter;
    private final AuthenticationProvider authenticationProvider;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeHttpRequests()
            .requestMatchers("/api/auth/**").permitAll()
            .anyRequest().authenticated()
            .and()
            .sessionManagement()
            .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
            .authenticationProvider(authenticationProvider)
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);
        return http.build();
    }
}

В данном коде производится базовая настройка Spring Security. Отключается CSRF-защита, так как используется stateless-аутентификация. Все запросы к /api/auth/** доступны без авторизации (регистрация, логин). Остальные запросы требуют JWT-токен. Устанавливается политика STATELESS, чтобы сервер не хранил сессии. Добавляется фильтр jwtAuthFilter для проверки токена в каждом запросе.

@Component
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
                                    throws ServletException, IOException {

        final String authHeader = request.getHeader("Authorization");
        final String jwt;
        final String userEmail;

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }

        jwt = authHeader.substring(7);
        userEmail = jwtService.extractUsername(jwt);

        if (userEmail != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = this.userDetailsService.loadUserByUsername(userEmail);
            if (jwtService.isTokenValid(jwt, userDetails)) {
                UsernamePasswordAuthenticationToken authToken =
                        new UsernamePasswordAuthenticationToken(
                                userDetails,
                                null,
                                userDetails.getAuthorities()
                        );
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }
        filterChain.doFilter(request, response);
    }
}

Этот фильтр обрабатывает каждый входящий HTTP-запрос. Извлекает JWT из заголовка Authorization. Проверяет токен с помощью JwtService.
Если токен корректный – устанавливает контекст безопасности, предоставляя пользователю доступ к защищённым ресурсам.

@Service
public class JwtService {

    private static final String SECRET_KEY = "mySecretKeyExample123456";

    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    public <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }

    public String generateToken(UserDetails userDetails) {
        return Jwts.builder()
                .setSubject(userDetails.getUsername())
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60 * 24))
                .signWith(getSignInKey(), SignatureAlgorithm.HS256)
                .compact();
    }

    public boolean isTokenValid(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        return (username.equals(userDetails.getUsername())) && !isTokenExpired(token);
    }

    private boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }

    private Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }

    private Claims extractAllClaims(String token) {
        return Jwts.parserBuilder()
                .setSigningKey(getSignInKey())
                .build()
                .parseClaimsJws(token)
                .getBody();
    }

    private Key getSignInKey() {
        byte[] keyBytes = Decoders.BASE64.decode(SECRET_KEY);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}

JwtService отвечает за:
– генерацию токена при успешной аутентификации,
– валидацию токена,
– извлечение информации о пользователе (subject) и сроке действия.
Токен подписывается алгоритмом HS256 и содержит дату истечения срока действия.

@Service
@RequiredArgsConstructor
public class AuthenticationService {

    private final UserRepository repository;
    private final PasswordEncoder passwordEncoder;
    private final JwtService jwtService;
    private final AuthenticationManager authenticationManager;

    public AuthenticationResponse register(RegisterRequest request) {
        var user = User.builder()
                .username(request.getUsername())
                .password(passwordEncoder.encode(request.getPassword()))
                .role(Role.USER)
                .build();

        repository.save(user);
        var jwtToken = jwtService.generateToken(user);
        return AuthenticationResponse.builder()
                .token(jwtToken)
                .build();
    }
}

Здесь используется BCryptPasswordEncoder для безопасного хеширования паролей. Каждый пароль кодируется с использованием случайной «соли», что делает невозможным обратное восстановление или подбор хеша. После регистрации пользователь получает JWT для дальнейшего доступа.

@Configuration
public class PasswordConfig {
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
Компонент BCryptPasswordEncoder реализует алгоритм BCrypt. Он автоматически генерирует соль и позволяет регулировать уровень сложности хеширования через параметр «cost factor», обеспечивая высокий уровень защиты паролей.
В архитектуру программной системы добавлен слой безопасности (Security Layer), включающий:
– фильтр JwtAuthenticationFilter;
– сервис JwtService;
– менеджер аутентификации;
– хранилище пользователей и ролей.
Теперь взаимодействие между клиентом и сервером осуществляется через JWT, который клиент передаёт в каждом запросе. Сервер проверяет подпись токена и срок его действия, после чего предоставляет доступ к нужным ресурсам. Все данные передаются по защищённому протоколу HTTPS.
Реализованная система аутентификации и авторизации обеспечивает надёжный и гибкий механизм управления доступом.
Использование Spring Security и JWT позволяет достичь высокой степени защиты, а применение BCrypt обеспечивает безопасность хранения паролей.
Такая архитектура полностью удовлетворяет требованиям безопасности, масштабируемости и интеграции с внешними системами аутентификации.

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

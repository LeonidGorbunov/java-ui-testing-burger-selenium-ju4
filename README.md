[![UI Tests (Headless Chrome) & Allure Report](https://github.com/LeonidGorbunov/java-ui-testing-burger-selenium-ju4/actions/workflows/tests.yml/badge.svg)](https://github.com/LeonidGorbunov/java-ui-testing-burger-selenium-ju4/actions/workflows/tests.yml)
[![Allure Report](https://img.shields.io/badge/Allure%20Report-Live-brightgreen)](https://leonidgorbunov.github.io/java-ui-testing-burger-selenium-ju4/)
# Java UI Testing Stellar Burgers Project (Selenium)

**Контекст и цели проекта**: проектирование стабильной и легко поддерживаемой архитектуры фреймворка автоматизации пользовательских сценариев (UI Automation) на базе паттерна Page Object Model (POM) для верификации критических путей фронтенда космической бургерной и кроссбраузерной проверки рендеринга интерфейса.

**ТЗ (вид работ)**: автоматизация пользовательского интерфейса (Advanced UI Automation), разработка слоя Selenium-обёрток, проектирование параметризованной фабрики кроссбраузерного запуска (WebDriver Factory), построение гибридных UI+API фикстур данных.

**Продукт**: веб-интерфейс фронтенда сервиса Stellar Burgers.

**Выполненные задачи**:
1. **Page Object Architecture**: разработан абстрактный базовый класс `BasePage` с инкапсуляцией явных ожиданий `WebDriverWait` (`ExpectedConditions.elementToBeClickable`, `visibilityOfElementLocated`). Все атомарные операции взаимодействия с элементами снабжены явными ожиданиями и полностью скрыты внутри обёрток `click`, `type` и `getText`, что ликвидирует риски падения тестов по исключениям `StaleElementReferenceException`. Наследники страниц (`MainPage`, `LoginPage`, `RegisterPage`, `ForgotPasswordPage`) оперируют бизнес-сценариями высокого уровня абстракции.
2. **Кроссбраузерная Фабрика Драйверов (`WebDriverFactory`)**: спроектирован выделенный менеджер конфигураций браузеров на основе оператора `switch-case`. Реализована поддержка headless режима (`--headless=new`) с флагами оптимизации рендеринга (`--window-size=1920,1080`, `--disable-gpu`, `--no-sandbox`). Интегрирована автоматическая загрузка бинарников Chrome через `WebDriverManager` и кастомная инициализация Яндекс.Браузера посредством явной привязки локальных путей (`options.setBinary`).
3. **Advanced JavaScript Injection (Умный скроллинг)**: для преодоления нестабильности стандартных методов Selenium разработан метод контроля прокрутки `isContainerScrolledToElement`. Алгоритм использует кастомный скрипт JavaScript, вычисляет динамические координаты элемента контейнера меню через `getBoundingClientRect()` и возвращает булево значение успешности скролла этого элемента до верхней границы контейнера с погрешностью до 15 пикселей в рамках явного ожидания.
4. **Гибридные UI + API Cross-Layer Фикстуры**: для минимизации времени прохождения UI-тестов и обеспечения атомарности проверок внедрён сквозной подход. Предусловия авторизации (создание уникального юзера) выполняются мгновенно через фоновые HTTP-запросы `UserClient` (RestAssured Layer).
5. **Автономный Teardown (Self-Healing Teardown)**: очистка окружения в блоках `@After` делегирована отказоустойчивому сервису `UserTeardownSteps`. Метод снабжён алгоритмом преавторизации на бэкенде: в случае отсутствия или сброса UI-токена в рантайме, сервис самостоятельно запрашивает новый токен по регистрационным данным и гарантированно удаляет сущность из БД.
6. **BDD-Трассировка Allure**: фреймворк размечен аннотациями `@Epic`, `@Feature`, `@DisplayName` и `@Description`. Каждое низкоуровневое действие и шаг верификации снабжены аннотациями `@Step` с поддержкой динамической подстановки локаторов для полной прозрачности логов выполнения.

**Область тестирования (Scope)**: сквозные цепочки успешной авторизации через 4 различных UI-точки входа; валидация форм регистрации и перехват локальных системных ошибок фронтенда на длину пароля; верификация функциональности вкладок Конструктора ингредиентов с математическим контролем скролла ленты меню.

**Окружение**: Java (JDK 11), JUnit 4, Selenium WebDriver, WebDriverManager, RestAssured, Lombok, GSON, Allure Reporting, AspectJ Weaver Agent, Maven.


### Визуализация тестового репортинга Allure & Cross-Browser Architecture

#### 1. Общий дашборд кроссбраузерного выполнения (Overview Dashboard)
Финальный срез выполнения автоматизированного UI тест-комплекта. Демонстрирует одновременную агрегацию результатов кроссбраузерного прогона (18 тест-кейсов) и интеграцию мета-параметров среды запуска (виджет Environment) через автоматизированный сборщик `maven-resources-plugin`.
![Allure UI Dashboard Overview](allure-ui-overview.png)

#### 2. Динамическое разделение сред выполнения (Advanced BDD Chrome Behaviors)
Результат проектирования кастомной архитектуры `BaseTest` на базе `Parameterized.class`. Подстановка мета-параметров (`@Parameterized.Parameters`) заставила Allure JUnit 4 генерировать уникальные хэши `historyId` для каждого окружения. Это позволило полностью избежать схлопывания отчётов во вкладку Retries и вывело дерево тестов в обоих окружениях в левую панель Behaviors.
![Allure UI Chrome Behaviors](allure-ui-behaviors.png)

#### 3. Верификация сложных UI-компонентов (Yandex Browser Scroll Execution)
Успешное выполнение теста на переключение табов и валидацию скроллинга раздела Конструктор в Яндекс.Браузере. Скриншот демонстрирует прохождение теста благодаря инжекции JavaScript-алгоритма `isContainerScrolledToElement` на базе вычисления координат `getBoundingClientRect()`.
* **Анализ инфраструктурной нестабильности стенда (Root Cause Analysis)**: В ходе headless-прогона в окружении Yandex Browser были зафиксированы два тест-кейса в статусе `Broken` по причине `org.openqa.selenium.TimeoutException: timeout: Timed out receiving message from renderer`. Данный сбой носит исключительно инфраструктурный характер (Flaky Environment) и вызван задержками рендеринга на стороне слабого тестового сервера.
![Allure UI Yandex Scroll](allure-ui-yandex-scroll.png)

### 📊 Живой отчёт Allure и запуск в CI/CD (GitHub Actions)

* **Интерактивный отчёт последнего прогона (Headless Chrome):** [Открыть Allure Report на GitHub Pages](https://leonidgorbunov.github.io/java-ui-testing-burger-selenium-ju4/)
* **Особенности запуска в облаке:** в репозиторий интегрирован пайплайн [GitHub Actions](.github/workflows/tests.yml), который запускает тесты на Linux в режиме `Headless Chrome` (`-Dheadless=true -Dbrowser=chrome`), сохраняет историю прогонов и публикует отчёт на GitHub Pages. Полный кроссбраузерный прогон с **Яндекс.Браузером** поддерживается при локальном запуске на Windows.

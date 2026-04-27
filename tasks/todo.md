# tasks/todo.md — Price Monitor v2.0 bootstrap plan

## Специфікація
- **Вхід:** технічне завдання Price Monitor v2.0 (UA), production baseline 2026-04-27.
- **Вихід:** репозиторний baseline-артефакт специфікації + стартовий план реалізації/верифікації.
- **Інваріанти:** мінімальний diff, без ризикових дій, жодних неперевірених тверджень.
- **Stop-тригери:** відсутність критеріїв успіху, конфлікт вимог, неможливість верифікації.

## План
- [x] Створити baseline-документ у репозиторії (`docs/price-monitor-v2-baseline.md`).
- [x] Зафіксувати tasks/todo з критеріями, межами та verification plan.
- [ ] Узгодити пріоритети імплементації (Phase 1 scope split на інкременти).
- [ ] Додати ADR для архітектурних рішень (ETL orchestration, evidence retention, API contracts).
- [ ] Реалізувати кістяк сервісу та схему БД з міграціями.
- [ ] Реалізувати модулі Scraper/Verify/Audit/Pricing + unit/integration tests.

## Verification plan
1. Лінтер/форматування markdown (опційно).
2. Перевірка наявності файлів і очікуваних секцій.
3. Git diff review: лише doc/task зміни.

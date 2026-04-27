# Price Monitor v2.0 — Production Baseline (UA)

**Версія документа:** 1.0  
**Дата:** 2026-04-27  
**Статус:** Production Baseline

## Призначення
Цей документ фіксує технічний baseline для системи моніторингу верифікованих цін України
(Price Monitor v2.0) та використовується як канонічна специфікація для реалізації і валідації.

## Ключові інваріанти (non-negotiable)
- Зберігати порядок рядків Excel (`row_index`) у виході.
- Валюта виводу — завжди UAH.
- Історичні ціни не видаляються (лише versioning).
- Заборонено anti-bot bypass / CAPTCHA solving.
- Ціна без MHTML evidence не використовується в `expected_price`.
- Обовʼязкове дотримання `robots.txt`.

## Архітектурний потік
`Excel → Search → Fetch → Verify → Audit → Output`

Підсистеми: API Gateway, Orchestrator, Search, Fetch, Verify, Audit, Excel OutputWriter,
Analytics (LightGBM/SHAP, Prozorro Comparison).

## Ключові модулі
1. **Scraper + ComplianceAgent**: robots.txt check, rate limiting, fallback (requests → Playwright → cache).
2. **AnomalyDetector (L1–L5)**: Bounds, Flash Crash, Z-score, CUSUM, Cross-source consensus.
3. **SpecMatcher**: `exact/near/mismatch/unknown`, `spec_match_score` 0..1, `critical_spec_mismatch`.
4. **AuditEngine**: flags, `priority_score`, `risk_level`.
5. **PricingEngine**: median/average/single/no_data + versioning `expected_prices`.
6. **PDF Evidence (6 сторінок)**: ринкові дані, Prozorro порівняння, журнал аномалій, chain of evidence.
7. **Prozorro Integration**: API-only, materialized view `mv_price_comparison`.

## Acceptance Criteria (узагальнено)
- Anomaly rules працюють за порогами та правильно мапляться в `ACCEPT/QUARANTINE/REJECT`.
- Pricing враховує лише valid+evidence offers та versioning `expected_prices`.
- `POST /api/v1/audit-pdf/{run_item_id}` генерує 6-сторінковий PDF.
- Prozorro comparison повертає `spread_pct` і `spread_signal`.
- `GET /api/v1/prices/{sku}/expected` відповідає в p95 < 200 ms.
- Excel output зберігає початковий порядок рядків.

## API baseline
- `GET /health`
- `GET /api/v1/prices/{sku}/expected`
- `GET /api/v1/prices/{sku}/comparison`
- `POST /api/v1/audit-pdf/{run_item_id}`
- `GET /api/v1/runs/`
- `GET /api/v1/audit/flags`
- `POST /api/v1/recalculate/{sku}`

## Дані та схема
Основні таблиці: `products`, `sources`, `runs`, `run_items`, `offers`, `audit_flags`,
`evidence_files`, `historical_prices`, `expected_prices`, `procurement_history`.

Ключові вимоги:
- `products.sku` — UNIQUE
- `expected_prices` — пошук актуальної версії через `valid_to IS NULL`
- `offers` індексується за `(item_id, created_at)` та `is_valid`

## Roadmap
- **Phase 1 (MVP Stabilization):** anomaly/spec/audit/pricing/pdf/prozorro baseline.
- **Phase 2 (Intelligence):** LightGBM + SHAP + LME features.
- **Phase 3 (Microservices):** Celery + Kubernetes + alerting.
- **Phase 4 (Dashboard):** GUI + KPI exporter + розширена аналітика.

## Примітка
Повна розширена специфікація (детальні FR/NFR/ETL/алгоритми/Legal/Risk Register) є
канонічним джерелом вимог і повинна використовуватися під час імплементації та ревʼю.

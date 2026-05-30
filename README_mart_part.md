# Проект: Анализ ДТП Москвы 2020–2024

Аналитическая система для исследования факторов дорожно-транспортных происшествий.
Данные обогащены погодными, пространственными и поведенческими признаками.
Финальные витрины готовы для загрузки в Yandex DataLens.

---

## Структура папок

```
transport_analys/
├── input/                               # Исходные Excel-файлы (не редактировать!)
│   ├── 2020_выгрузка.xlsx
│   ├── 2021_выгрузка.xlsx
│   ├── 2022_выгрузка.xlsx
│   ├── 2023_выгрузка.xlsx
│   └── 2024_выгрузка.xlsx
│
├── kley.ipynb                           # Студент 1: сборка базовых таблиц
├── weather_feature_modified.ipynb       # Студент 2: погодное обогащение
├── spatial_features.ipynb               # Студент 3: пространственные признаки
├── build_marts.ipynb                    # Студент 4: витрины + ML-модель
│
├── output/                              # Базовые таблицы (Студент 1)
│   ├── fact_dtp.csv                     # 41 131 строка
│   ├── fact_participant.csv             # 98 402 строки
│   ├── fact_vehicle.csv                 # 71 589 строк
│   ├── data_dictionary.xlsx             # Словарь полей
│   └── coverage_report.txt              # Отчёт покрытия
│
├── output_w/                            # Погодные признаки (Студент 2)
│   └── feat_weather_dtp_first4600lines.csv  # Покрытие ~11% (4 600 строк)
│                                            # ⚠ Тестовый прогон, ждём полный
│
├── output_spatial/                      # Гео-признаки (Студент 3)
│   ├── feat_spatial_dtp.csv             # Покрытие ~60% (24 552 строки)
│   ├── agg_cell_space.csv               # Агрегаты по H3-ячейкам
│   ├── osm_cell_cache.json              # Кэш OSM-запросов
│   ├── coverage_report.txt              # Отчёт покрытия
│   └── README.md                        # Описание методики геообогащения
│
├── output_mart/                         # Финальные витрины (Студент 4)
│   ├── mart_dtp_enriched.csv            # Главная витрина: всё в одной таблице
│   ├── mart_factor_profile.csv          # Флаги факторов риска для DataLens
│   ├── mart_dashboard_overview.csv      # Агрегаты по году/месяцу/округу
│   ├── mart_time_dynamics.csv           # Временная динамика
│   ├── mart_spatial_risk.csv            # Агрегаты по H3-ячейкам для карты
│   ├── mart_weather_context.csv         # Погодные агрегаты по сезонам
│   ├── mart_cell_day.csv                # Детализация по ячейке + день недели
│   ├── mart_cell_hour.csv               # Детализация по ячейке + час
│   ├── feature_importance.csv           # Важность признаков (ML)
│   └── factor_dictionary.xlsx           # Паспорт всех признаков
│
└── reports/
    ├── final_quality_report.xlsx        # Сводный отчёт качества
    └── feature_importance.png           # График топ-15 факторов
```

---

## Порядок запуска

Ноутбуки запускаются последовательно — каждый следующий зависит от результатов предыдущего.

```
1. kley.ipynb                      → output/fact_*.csv
2. weather_feature_modified.ipynb  → output_w/feat_weather_dtp*.csv
3. spatial_features.ipynb          → output_spatial/feat_spatial_dtp.csv
4. build_marts.ipynb               → output_mart/* + reports/*
```

---

## Финальные витрины

| Файл | Строк | Описание |
|---|---|---|
| `mart_dtp_enriched.csv` | 41 131 | Главная витрина: ДТП + погода + гео + классификаторы + risk_score |
| `mart_factor_profile.csv` | 41 131 | Флаги факторов в формате "Да"/"Нет" для DataLens |
| `mart_dashboard_overview.csv` | 8 011 | Агрегаты по году/месяцу/округу/типу ДТП |
| `mart_time_dynamics.csv` | 560 | Временная динамика с долями нарушений |
| `mart_spatial_risk.csv` | 4 913 | Агрегаты по H3-ячейкам — для карты рисков |
| `mart_weather_context.csv` | 61 | Погодные агрегаты по годам и сезонам |
| `mart_cell_day.csv` | 23 582 | Детализация рисков по ячейке + день недели |
| `mart_cell_hour.csv` | 20 547 | Детализация рисков по ячейке + час |
| `feature_importance.csv` | 30 | Важность признаков из Random Forest |

---

## ML-модель

**Задача:** предсказание тяжести ДТП (severity: 1 = гибель, 0 = ранение)  
**Алгоритм:** Random Forest (200 деревьев, max_depth=10, class_weight=balanced)  
**Метрика:** ROC-AUC = **0.762**

### Топ-7 факторов по важности:

| Место | Признак | Важность | Интерпретация |
|---|---|---|---|
| 1 | type_code | 0.186 | Тип ДТП — главный предиктор тяжести |
| 2 | hour | 0.137 | Час суток: ночные ДТП смертельнее |
| 3 | has_truck | 0.108 | Грузовик резко повышает летальность |
| 4 | is_night_dtp | 0.095 | Ночное время суток |
| 5 | distance_to_bus_stop_m | 0.084 | ДТП далеко от остановок — открытые участки |
| 6 | has_distance_viol | 0.060 | Нарушение дистанции |
| 7 | has_speed_violation | 0.049 | Нарушение скорости |

### Формула risk_score:

```
risk_score = (
    type_code              × 0.258 +
    hour                   × 0.191 +
    has_truck              × 0.150 +
    is_night_dtp           × 0.132 +
    distance_to_bus_stop_m × 0.117 +
    has_distance_viol      × 0.083 +
    has_speed_violation    × 0.068
) → нормировано в [0, 1]
```

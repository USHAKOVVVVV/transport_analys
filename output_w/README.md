readme_content = """# Обогащение данных о ДТП погодными, астрономическими и календарными признаками

## Источники данных

| Признак | Источник | Описание |
|---------|----------|----------|
| Погодные (temp_c, feels_like_c, precipitation_type, precipitation_mm, snow_depth_cm, wind_speed_ms, wind_gust_ms, humidity_pct, pressure_hpa, visibility_m) | Open-Meteo Historical API (https://open-meteo.com/en/docs/historical-weather-api) | Данные реанализа ERA5, почасовые, привязаны к координатам ДТП. Видимость (visibility_m) не предоставляется API – поле заполнено NaN. |
| Астрономические (sunrise_ts, sunset_ts, daylight_minutes, is_twilight, is_dark, is_daylight) | Библиотека `astral` (https://astral.readthedocs.io/) | Расчёт времени восхода/заката и сумерек по координатам и дате. |
| Календарные (day_of_week, is_weekend, is_holiday, holiday_name, season) | Библиотека `holidays` (https://github.com/vacanza/python-holidays) + ручные вычисления | Государственные праздники РФ, выходные дни, сезон года. |
| Производные индикаторы (ice_risk, low_visibility_flag, bad_weather_flag, dark_without_lights_flag) | Вычислены на основе погодных, астрономических и исходных данных | Правила расчёта описаны ниже. |

## Карта соответствия «что было – что добавили»

| Было (исходные поля ДТП) | Добавлено (новые поля) | Единица измерения |
|--------------------------|------------------------|-------------------|
| moment_date, moment_time | temp_c | °C |
| place_latitude, place_longitude | feels_like_c | °C |
| district_id, area_id | precipitation_type | текст (rain/snow/none) |
| light_type_code | precipitation_mm | мм |
| meteo_clouds | snow_depth_cm | см |
| | wind_speed_ms | м/с |
| | wind_gust_ms | м/с |
| | humidity_pct | % |
| | pressure_hpa | гПа |
| | visibility_m | м (всегда NaN, т.к. API не отдаёт) |
| | sunrise_ts | timestamp |
| | sunset_ts | timestamp |
| | daylight_minutes | минуты |
| | is_twilight | 0/1 (флаг) |
| | is_dark | 0/1 (флаг) |
| | is_daylight | 0/1 (флаг) |
| | day_of_week | 0-6 (пн=0) |
| | is_weekend | 0/1 |
| | is_holiday | 0/1 |
| | holiday_name | текст |
| | season | winter/spring/summer/autumn |
| | ice_risk | 0/1 |
| | low_visibility_flag | 0/1 |
| | bad_weather_flag | 0/1 |
| | dark_without_lights_flag | 0/1 |

## Правила расчёта производных индикаторов

- **ice_risk** = 1, если температура от -2 до +2°C **И** (осадки > 0 ИЛИ высота снега > 0). Иначе 0.
- **low_visibility_flag** = 1, если видимость < 1000 м. (Всегда 0, т.к. видимость отсутствует.)
- **bad_weather_flag** = 1, если (скорость ветра > 15 м/с) ИЛИ (осадки > 5 мм) ИЛИ (видимость < 500 м).
- **dark_without_lights_flag** = 1, если `is_dark = 1` **И** в исходном поле `light_type_code` присутствует фраза "освещение не включено".

## Привязка погодных данных к ДТП

Для каждого ДТП запрашивались почасовые исторические данные с API Open-Meteo по точным координатам места происшествия. Затем выбирался час, ближайший к моменту ДТП (по минимальной разнице во времени).

## Повторный пересчёт

Все признаки могут быть пересчитаны путём повторного запуска скрипта (ноутбука). Для этого необходимо:
1. Сохранить исходный `.csv` в рабочей директории.
2. Выполнить все ячейки в порядке, описанном в решении.
3. На выходе получатся файлы `feat_weather_dtp.csv` и `weather_cell_time.csv`.
"""


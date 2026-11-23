# Прогнозирование реализованной волатильности: HAR-RV vs градиентный бустинг

Прогноз дневной реализованной волатильности (log-RV) на следующий торговый день для шести рядов: золото, ОФЗ, USD/RUB, ВТБ (2016–2021 и 2011–2022), биткоин. Опционально подгружаются свежие данные ВТБ через API Мосбиржи (MOEX ISS).

**Главный вопрос:** дают ли CatBoost, LightGBM и их стекинг прирост качества по сравнению с классической эконометрической моделью **HAR-RV** (Corsi, 2009) при честной out-of-sample проверке?

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/itsmyhomeee/volatility-forecasting/blob/main/volatility_forecasting.ipynb)

## Подход

**Данные.** Дневные OHLCV-котировки (выгрузки ФИНАМ из репозитория [5-diff.-quotes](https://github.com/itsmyhomeee/5-diff.-quotes)) и, опционально, MOEX ISS API. Дни без внутридневного диапазона (High = Low) удаляются, OHLC проверяются на согласованность, скачки доходности > 50% за день (сплиты) обнуляются.

**Целевая переменная.** log RV следующего дня, RV — оценка Garman–Klass по OHLC.

**Признаки.** HAR-компоненты (дневной, недельный и месячный log-RV), leverage-эффект, ночной гэп, тренд и волатильность волатильности, относительный объём, день недели. Все признаки используют только информацию, доступную на закрытие дня *t*.

**Модели.**

| Модель | Описание |
|---|---|
| Naive | RV завтра = RV сегодня |
| HAR-RV | OLS по трём HAR-компонентам — эталон |
| CatBoost, LightGBM | гиперпараметры: `RandomizedSearchCV` + `TimeSeriesSplit` |
| HAR + LGBM | LightGBM прогнозирует остатки HAR |
| Среднее | HAR, CatBoost, LightGBM |
| Стекинг | линейная мета-модель с неотрицательными весами на out-of-fold прогнозах |

**Валидация.** Хронологическое разбиение (последние 20% — тест), подбор гиперпараметров только на train, walk-forward проверка на пяти последовательных окнах.

**Метрики.** RMSE, MAE, R² (log-RV), **QLIKE** (стандартная функция потерь для волатильности), прирост относительно HAR в процентах, **тест Диболда–Мариано** с HAC-дисперсией. Интерпретация — SHAP-значения CatBoost.

## Результаты

> Таблицы, графики и выводы формируются в ноутбуке автоматически по итогам запуска.

<!-- Вставьте сюда итоговую таблицу RMSE / QLIKE и ключевой вывод после запуска -->

## Что изменено по сравнению с первой версией (курсовой проект)

- случайное `train_test_split` заменено хронологическим разбиением — убрана утечка будущего;
- удалены дни без торгов, которые давали log-RV ≈ −18 и искажали MAE/RMSE;
- добавлены эталон HAR-RV, QLIKE и тест Диболда–Мариано;
- стекинг переведён на out-of-fold прогнозы по времени;
- добавлены модель «HAR + бустинг на остатках», walk-forward и SHAP;
- выводы формируются из фактических результатов, а не пишутся заранее.

## Запуск

```bash
pip install -r requirements.txt
jupyter notebook volatility_forecasting.ipynb
```

Или откройте ноутбук в Google Colab и выполните *Runtime → Run all*. Данные подгружаются автоматически, полный прогон занимает около 10–15 минут (параметр `N_ITER` регулирует глубину подбора гиперпараметров).

## Стек

Python · pandas · NumPy · scikit-learn · CatBoost · LightGBM · SciPy · Matplotlib · Seaborn · MOEX ISS API

## Литература

- Corsi F. (2009). *A Simple Approximate Long-Memory Model of Realized Volatility.* Journal of Financial Econometrics.
- Garman M., Klass M. (1980). *On the Estimation of Security Price Volatilities from Historical Data.* Journal of Business.
- Patton A. (2011). *Volatility Forecast Comparison Using Imperfect Volatility Proxies.* Journal of Econometrics.
- Diebold F., Mariano R. (1995). *Comparing Predictive Accuracy.* Journal of Business & Economic Statistics.

---
Автор: Евгений Макаров · [Telegram](https://t.me/itsmyhm)

# Crypto Move Readiness Scanner

Скрипт автоматически:
1) Берёт топ-N монет по капитализации с CoinGecko  
2) Тянет OHLCV c крупных бирж через CCXT (приоритет: Binance → OKX → Coinbase)  
3) Считает индикаторы (RSI, MFI, Bollinger width, ATR%, Volume Z-score, 24h change)  
4) Пытается получить текущую ставку фондирования с Binance USDT‑m фьючерсов  
5) Строит Композитный "Move Readiness Score" — чем больше по модулю, тем выше вероятность сильного движения  
6) Пишет Excel-отчёт: лист **Summary** (ранжирование) + по одному листу на каждый актив (последние свечи с индикаторами)

## Установка

```bash
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS/Linux:
source .venv/bin/activate

pip install -r requirements.txt
```

## Запуск

```bash
python main.py --tf 1h --days 90 --top 50 --outfile crypto_move_readiness_report.xlsx
```

Параметры:
- `--tf` таймфрейм: `1m,5m,15m,1h,4h,1d` (в коде задано соответствие)  
- `--days` сколько дней истории тянуть (для 1h/4h достаточно 60–180 дней)  
- `--top` сколько монет с CoinGecko брать (по умолчанию 50)  
- `--outfile` имя Excel-файла (по умолчанию `crypto_move_readiness_report.xlsx`)

## Как интерпретировать Score

Score строится из нормализованных компонентов:

- RSI (дистанция от 50)  
- MFI (дистанция от 50)  
- Ширина полос Боллинджера (шире → выше потенциал движения)  
- Volume Z-score (всплеск объёма)  
- ATR% (текущая волатильность)  
- Ставка фондирования (крайние значения могут указывать на перекос)

Весовые коэффициенты заданы в коде (см. `weights`). Положительный Score чаще совпадает с бычьим импульсом, отрицательный — с медвежьим или риском "squeeze" в противоположную сторону. Рекомендуется дополнительно смотреть **abs_score** — величину импульса.

## Замечания

- Для некоторых монет на выбранных биржах может не быть данных — такие активы пропускаются.  
- Ставка фондирования берётся с Binance USDT‑m фьючерсов, если контракт PERP есть. Если нет — поле пустое.  
- Если вы хотите учитывать open interest, basis, funding history, опционы — можно расширить код (Coinglass/Laevitas/Deribit API и т.п.).  
- Для стабильных монет (USDT/USDC/FDUSD/DAI/TUSD) данные не считаются и они фильтруются.

## Что именно писать в Excel

Лист **Summary**:
- `asset, exchange, symbol, timestamp, close, volume, rsi14, mfi14, bb_width, atr_pct, vol_z, chg_24, funding_rate, move_readiness_score, abs_score`  
Отсортировано по `move_readiness_score` (по убыванию).

Остальные листы содержат последние ~300 свечей с рассчитанными индикаторами.

Удачи!
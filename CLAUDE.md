# Apex Claude Trader — Estado da Sessao

## MODO ATUAL: AUTO-TRADE (v3)
Bot analiza mercado, gera sinais E executa ordens automaticamente via Playwright (browser fetch).
Config: AUTO_EXECUTE_TRADES = True em config/settings.py

## ARQUITETURA DE ORDENS
- Autenticacao: Playwright Chromium stealth (data/browser_executor)
- Envio: page.evaluate() com fetch() DENTRO do navegador autenticado
- API direta: REVOGADA ("The app is not registered")
- Ordens vao para: demo.tradovateapi.com/v1/order/placeOSO
- бракет: Stop + Limit automaticos

## CONTA TRADOVATE
- Usuario: APEX_549417 | Conta demo: APEX5494170000001 (ID: 45402487)
- Renovação saldo mesa proprietária: 01/05/2026
- Base DEMO: https://demo.tradovateapi.com/v1
- Base LIVE: https://live.tradovateapi.com/v1
- Config: `config/settings.py`

## ARQUIVOS PRINCIPAIS
- `main.py` — Modo auto-trade, executor automatico
- `core/auth.py` — TradovateAuth (async)
- `core/market_data.py` — Candles via yfinance
- `core/risk.py` — Risk management
- `core/executor.py` — Executor ordens via browser fetch (Playwright)
- `skills/strategy.py` — Motor multi-estrategia
- `skills/adaptive_threshold.py` — Threshold dinamico
- `skills/news_filter.py` — Filtro noticias/volatilidade

## CONFIGS
- SYMBOL=ES | TIMEFRAME=5min | Min Score=4 (adaptativo)
- AUTO_EXECUTE_TRADES=True
- Horario: 9h-21h BRT | Max daily loss=$800 | Max daily profit=$600
- Stop=16 ticks | TP=40 ticks | R/R 1:2.5

## DADOS
- `data/signals.json` — Registro de sinais
- `data/trading_journal.json` — Historico de trades
- `data/recovery_state.json` — Estado de recovery
- `data/token.json` — Token cache

## IMPORTANTE PARA PROXIMAS SESSOES
1. Bot JA faz ordens automaticas via Playwright (NÃO é modo sinais)
2. Executor usa page.evaluate(fetch()) dentro do browser autenticado
3. Se falhar, verificar logs/trader.log por erros de "Executor"
4. Sinal sonoro: `winsound.Beep()` no main.py
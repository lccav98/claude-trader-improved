# Apex Claude Trader — Recovery Guide

## Diagnóstico
Se o bot parar, verifique os logs primeiro:
```
tail -20 logs/trader.log
```

## Causas Comuns e Solução

### 1. Token Tradovate expirou (~59 min)
O bot tem auto-renew a cada 5 min. Se falhar:
- O bot tenta re-autenticar via API direta primeiro
- Depois via Chrome CDP (porta 9222)
- Depois via Chromium stealth com Playwright

Solução manual: abra `https://trader.tradovate.com`, faça login, o bot detecta o token na próxima renovação.

### 2. Feed de dados yfinance falhou
O market_data usa yfinance polling (não mais WebSocket). Se falhar:
- Verifique: `pip show yfinance` — precisa estar instalado
- Se necessário: `pip install yfinance --upgrade`

### 3. PyCache desatualizado (código antigo rodando)
Se log mostrar "WS desconectado" ou "md/subscribeChart":
```
cd "C:\Users\OpAC BV\Documents\Tradovate-Claude Code\apex-claude-trader"
rm -rf core/__pycache__ __pycache__ config/__pycache__ skills/__pycache__
python main.py
```

### 4. Processos Python duplicados
Verifique: `tasklist | grep python`
Mate duplicatas: `taskkill //F //IM python.exe` (mata todos)

### 5. Horário fora de operação
O bot roda 24h mas analisa sinais só entre 09h-17h BRT (configurável em settings.py).

---

## Estado Atual (Última Atualização: 2026-04-08 13:56 BRT)

### Modo: AUTO-TRADE
O bot executa ordens automaticamente via Playwright (browser fetch).
- AUTO_EXECUTE_TRADES = True em config/settings.py
- Não é mais modo sinais!

### Arquitetura
- **Feed de dados**: yfinance polling
- **Autenticação**: Playwright Chromium stealth (data/browser_executor)
- **Execução de ordens**: page.evaluate(fetch()) DENTRO do navegador autenticado
- **API direta**: REVOGADA ("The app is not registered")
- **Estratégias**: Bellinger Bands + Stochastic (6 estratégias votando, mínimo 3 para executar)

### Conta
- Conta: APEX5494170000001 (ID: 45402487)
- Ambiente: demo
- Instrumento: ES (anteriormente MNQ)
- Timeframe: 5 minutos
- Contracts: 1 por trade

### Config
- ENV: demo
- SYMBOL: ES
- TIMEFRAME_MINUTES: 5
- MAX_DAILY_LOSS: $800
- MAX_DRAWDOWN: $3000
- STOP_LOSS_TICKS: 12 (ou 16)
- TAKE_PROFIT_TICKS: 24 (ou 40)
- MIN_SCORE_TO_TRADE: 4 (settings.py) / 3 (runtime check)
- AUTO_EXECUTE_TRADES: True

### Arquivos Importantes
- `main.py` — ponto de entrada, modo auto-trade
- `core/market_data.py` — feed yfinance
- `core/auth.py` — autenticação Tradovate
- `core/executor.py` — envio de ordens via Playwright browser fetch
- `core/risk.py` — gerenciamento de risco
- `config/settings.py` — configurações
- `data/token.json` — token cache (expira em ~59 min)
- `data/trading_journal.json` — journal de trades

### Comandos Úteis
```
# Iniciar bot
cd "C:\Users\OpAC BV\Documents\Tradovate-Claude Code\apex-claude-trader"
python main.py

# Ver log em tempo real
Get-Content logs/trader.log -Wait -Tail 20

# Ver últimas linhas
Get-Content logs/trader.log -Tail 30

# Matar bot
taskkill //F //IM python.exe
```
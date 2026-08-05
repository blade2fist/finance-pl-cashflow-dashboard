# Методология

## Данные

Синтетическая (тестовая) компания с тремя бизнес-юнитами: Retail, Wholesale, E-commerce,
плюс корпоративный уровень (Corporate) для D&A, процентов по долгу и налога.
Период: январь 2023 — декабрь 2025 (36 месяцев).

Данные сгенерированы с реалистичными допущениями:
- рост выручки 0.4–1.4% в месяц в зависимости от бизнес-юнита;
- сезонность: провал в январе-феврале, пик в ноябре-декабре;
- COGS 52–66% от выручки в зависимости от юнита;
- налог на прибыль ~20% от EBT;
- квартальные транши долга (март/сентябрь) и его плановое погашение;
- дивиденды раз в квартал в размере 90% от чистой прибыли квартала.

## Логика формул в Excel

**P&L (P&L_Statement):**
```
Total Revenue = SUMIFS(Raw_Data, LineItem="Revenue")
Total COGS    = SUMIFS(Raw_Data, LineItem="COGS")
Gross Profit  = Total Revenue - Total COGS
Total Opex    = SUMIFS(Raw_Data, LineItem="Opex")
EBITDA        = Gross Profit - Total Opex
EBIT          = EBITDA - D&A
EBT           = EBIT - Interest Expense
Net Income    = EBT - Income Tax Expense
```

**Cashflow (Cashflow_Statement), связь с P&L:**
```
Net Income  -> ссылка на 'P&L_Statement' (зелёный шрифт = cross-sheet link)
+ D&A       -> ссылка на 'P&L_Statement'
+ Δ AR, Δ AP, Δ Inventory -> SUMIFS из Raw_Data (Operating CF)
= Net Cash from Operating Activities

- Capex     -> SUMIFS из Raw_Data (Investing CF)
= Net Cash from Investing Activities

+ Debt Issuance - Debt Repayment - Dividends -> SUMIFS из Raw_Data (Financing CF)
= Net Cash from Financing Activities

Net Change in Cash = Operating + Investing + Financing
Ending Cash Balance = Beginning Cash Balance + Net Change in Cash
```

Начальный остаток денежных средств (Beginning Cash, январь 2023) — единственное
жёстко заданное допущение ($250,000), отмечено синим шрифтом и комментарием в файле.

## Почему DataLens, а не Excel-дашборд

Excel хорош для построения самой модели (формулы, аудируемость, контроль допущений).
Для интерактивного дашборда с фильтрами использован DataLens — он не открывает `.xlsx`
напрямую, поэтому расчётные данные выгружены в плоскую таблицу
(`Date | Metric | Value`) и загружены через Google Sheets.

## Шаги подключения DataLens → Google Sheets

1. Данные из `DataLens_Export` выгружены в Google Sheets (публичный доступ по ссылке).
2. DataLens: Создать → Подключение → Google Sheets → вставить ссылку на таблицу.
3. Создать Датасет: поле `Date` → тип Дата, `Value` → тип Дробное число (агрегация Сумма),
   `Metric` → тип Строка.
4. Чарты в режиме Wizard: `Date` → X, `Value` → Y (Сумма), `Metric` → Цвета/Фильтр.
5. Дашборд: 4 чарта + единый селектор-фильтр по полю `Date`, связанный со всеми чартами
   через раздел «Связи».

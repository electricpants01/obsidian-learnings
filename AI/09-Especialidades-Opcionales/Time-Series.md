# Time Series - Series Temporales

## Descripción
Time series analiza datos secuenciales: forecasting, anomaly detection, trend analysis. Técnicas: ARIMA, Prophet, LSTM, Transformers. Casos: stock prediction, demand forecasting, IoT sensors, weather. Libraries: statsmodels, Prophet, pmdarima, sktime.

## Ejemplo
```python
from prophet import Prophet
df = pd.DataFrame({'ds': dates, 'y': values})
model = Prophet()
model.fit(df)
future = model.make_future_dataframe(periods=30)
forecast = model.predict(future)
```
---
**Tiempo**: 3-4 semanas

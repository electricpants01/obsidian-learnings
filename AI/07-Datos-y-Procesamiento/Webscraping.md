# Web Scraping - Extracción de Datos Web

## Descripción

Web scraping extrae datos de websites para ML datasets. Libraries: BeautifulSoup (HTML parsing), Scrapy (framework completo), Selenium (JavaScript rendering), Requests (HTTP). Use cases: price monitoring, news aggregation, reviews sentiment, training data collection. Legal/ethical: respetar robots.txt, rate limiting, no sobrecargar servers. Alternativa: APIs cuando disponibles.

## Herramientas

**BeautifulSoup**: HTML parsing simple
**Scrapy**: Production web crawling
**Selenium**: JavaScript-heavy sites
**Requests**: HTTP requests

## Ejemplo

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd

# Simple scraping
url = "https://example.com/products"
response = requests.get(url)
soup = BeautifulSoup(response.content, 'html.parser')

products = []
for item in soup.find_all('div', class_='product'):
    name = item.find('h2').text
    price = item.find('span', class_='price').text
    products.append({'name': name, 'price': price})

df = pd.DataFrame(products)

# Scrapy spider
import scrapy

class ProductSpider(scrapy.Spider):
    name = 'products'
    start_urls = ['https://example.com/products']
    
    def parse(self, response):
        for product in response.css('div.product'):
            yield {
                'name': product.css('h2::text').get(),
                'price': product.css('span.price::text').get()
            }
```

---
**Tiempo**: 1-2 semanas

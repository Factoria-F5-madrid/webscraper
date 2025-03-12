## Django Scraper

## Primer paso: Scraper con comando

Te voy a guiar paso a paso en los comandos que tienes que ejecutar en tu consola. Damos por hecho que tienes python instalado. 

- `mkdir webscraper`
- `cd webscraper`
- touch `.gitignore`  (Crear readme)
- Con ayuda de esta página puedes crear gitignore configurando lo que necesitas: Windows y Django por ejemplo: https://www.toptal.com/developers/gitignore)
- Crear entorno virtual: `python3 -m venv venv` o `python -m venv venv`
- Levantar entorno virtual: En OSX: `source venv/bin/activate` en Bash: `source venv/Script/active en Bash`
- (En el entorno virtual) `pip install django`
- `django-admin startproject webscraper_project`
- Queda así. (Puedes instalar tree para ver la estructura del proyecto y ejecutar tree -I "venv" para verlo)

```
webscraper/                               # Carpeta raíz del proyecto
├── env/                           # Entorno virtual
├── requirements.txt               # Dependencias del proyecto
└── webscraper_project/      # Carpeta del proyecto Django
    ├── manage.py                  # Comandos principales de Django
    └── webscraper_project/  # Configuración interna de Django
        ├── __init__.py
        ├── settings.py
        ├── urls.py
        ├── asgi.py
        └── wsgi.py
```

- `pip install selenium` (Selenium It’s slower than requests and BeautifulSoup because it loads the entire browser)
- `pip install webdriver-manager`
- `pip freeze > requirements.txt`
- Para comprobar: `cat requirements.txt` 

- cd webscraper_project
- python3 manage.py startapp scraper (Una "app" en Django es un módulo que encapsula cierta funcionalidad de tu proyecto, como el web scraping en este caso)
- Añado scraper en setting.py
```bash
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'scraper',  # Tu nueva app
]
```

- Creo el modelo para guardar info en scraper. En el archivo scraper>models.py

```python
from django.db import models

# Create your models here.
class ScrapedData(models.Model):
    title = models.CharField(max_length=200)
    url = models.URLField()
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title
```

- python3 manage.py makemigrations # Donde esté manage
- python3 manage.py migrate

- Puedo comprobar la estructura en SQlite para ver que todo va bien. Por defecto Django trabaja con Sqlite podrias cambiarlo en settings.py

- Dentro de scraper: cd scraper
- Creo mkdir services (Un servicio es una función que podemos reutilizar siempre que necesitemos)
-  touch ``__init__.py`` (para que lo pille como módulo, tiene que estar dentro de services)
- touch scrape.py (dentro de services)
- Con este contenido

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

# Para Chrome
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from webdriver_manager.chrome import ChromeDriverManager

def scrape_website():
    # Configurar Selenium
    options = Options()
    options.add_argument('--headless')  # Ejecutar en modo headless
    options.add_argument('--no-sandbox')  # Requerido para algunos servidores
    options.add_argument('--disable-dev-shm-usage')  # Para evitar errores de memoria

    # 🔹 Aquí inicializamos correctamente `service`
    service = Service(ChromeDriverManager().install())

    # Para Chrome
    # Selenium Manager se encargará de descargar y gestionar el WebDriver
    #service = Service()  # No es necesario especificar el ejecutable
    driver = webdriver.Chrome(service=service, options=options)

    # Navegar al sitio web
    url = "https://jorgebenitezlopez.com"
    driver.get(url)
    print(driver.title)  
# Esperar a que los elementos estén presentes
    try:
        WebDriverWait(driver, 10).until(
            EC.presence_of_all_elements_located((By.CSS_SELECTOR, "h1"))
        )
        titles = driver.find_elements(By.CSS_SELECTOR, "h1")
        urls = driver.find_elements(By.CSS_SELECTOR, "a")
    except Exception as e:
        print("Error al encontrar los elementos:", e)
        driver.quit()
        return []

    scraped_data = []
    for title, link in zip(titles, urls):
        scraped_data.append({
            "title": title.text,
            "url": link.get_attribute("href"),
        })

    print("Scraped data:", scraped_data)  # Para depuración
    driver.quit()
    return scraped_data
```

- Repasa el scraper: crea navegador, carga página y saca datos...

- Ahora que tenemos el scraper vamos a crear un comando para activarlo. ¿Que es un comando? Generalmente disparamos acciones cuando una url recibe una petición; pero también podemos crear nuestros propios comandos para disparar acciones. 🎯

- Dentro de scraper creo management/commands y un archivo scrape.py  (Importante los ``___init__.py`` en management y commands). El contenido del comando es el siguiente:

```python
from django.core.management.base import BaseCommand
from scraper.services.scrape import scrape_website
from scraper.models import ScrapedData

class Command(BaseCommand):
    help = "Run the web scraper"
    # Hereda de BaseCommand, lo que permite que este comando sea ejecutable mediante python manage.py <nombre_comando>.

    def handle(self, *args, **kwargs):
        # Ejecuta función
        data = scrape_website()
        print("Scraped Data:", data)  # Agrega esta línea para depurar
        # Guarda
        for item in data:
            ScrapedData.objects.create(title=item["title"], url=item["url"])
        # Confirma
        self.stdout.write(self.style.SUCCESS("Scraping completed!"))

```
- Importa comandos y crea uno sobre BaseCommand, que en definitiva le pone nombre a una acción para poder llamarla
- Ejecutar comando:  python3 webscraper_project/manage.py scrape
- Opcionalmente: Verifico que en la bd está la información
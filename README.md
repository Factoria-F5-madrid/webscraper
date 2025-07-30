## Django Screper

## Tercer paso: Cronjob

Para automatizar la acción

- Crear un archivo cronfile. Este es un cron job en Linux, que programa la ejecución periódica de un script en Python: ``touch cronfile``
- ``>> /var/log/cron.log`` → Agrega la salida estándar (stdout) al archivo de log /var/log/cron.log (sin sobrescribirlo).

```bash
*/5 * * * * /usr/local/bin/python /app/webscraper_project/manage.py scrape >> /var/log/cron.log 2>&1

```
- Comentario: >> /var/log/cron.log Agrega la salida estándar (stdout) del comando al archivo /var/log/cron.log, sin sobrescribirlo
- Comentario: Cuando editas un archivo cron (ya sea con crontab -e o importando un archivo con crontab cronfile), es importante que el archivo termine con un salto de línea. El salto de línea final indica que la última línea del cron está completa y debe ser interpretada. Si falta, el último cron job podría no registrarse ni ejecutarse correctamente.

- En el docker file

```bash
# Instalar cron y herramientas adicionales
RUN apt-get update && apt-get install -y \
    cron \
    vim \
    && rm -rf /var/lib/apt/lists/*

# Crear directorios necesarios y asignar permisos para cron
RUN mkdir -p /var/run /var/log && \
    chmod 0755 /var/run /var/log && \
    touch /var/run/crond.pid && \
    chmod 0644 /var/run/crond.pid && \
    touch /var/log/cron.log && \
    chmod 0644 /var/log/cron.log

# Copia el archivo local cronfile (que contiene las reglas de cron) al directorio /etc/cron.d/ con el nombre scrape-cron.
COPY cronfile /etc/cron.d/scrape-cron

# Dar permisos adecuados al cronfile
RUN chmod 0644 /etc/cron.d/scrape-cron

# Registrar el cronjob
RUN crontab /etc/cron.d/scrape-cron
```

- y cambio el final de docker file para que ejecute el comando del docker

```bash
CMD ["cron", "-f"]
```

- Para listarlo:

```bash
docker ps
docker exec -it webscraper-server-1 crontab -l
docker exec -it webscraper-server-1 service cron status

docker exec -it name bash
cd.. (A directorio raiz)
cat /var/log/cron.log
```

- Ejecutar el comando: docker exec -it webscraper-server-1 /usr/local/bin/python webscraper_project/manage.py scraper
- Comprobar la BBDD con antes


# Usa una imagen base de Ubuntu
FROM ubuntu:latest

# Instala Apache
RUN apt-get update && apt-get install -y apache2 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Expone el puerto 80 para Apache
EXPOSE 80

# Comando para iniciar Apache
CMD ["apache2ctl", "-D", "FOREGROUND"]

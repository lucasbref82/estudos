FROM ubuntu
RUN apt-get update && apt install curl -y
RUN useradd lucasbrenner
WORKDIR /app
COPY --chown=lucasbrenner:lucasbrenner --chmod=777 ./arquivo.txt .
USER lucasbrenner
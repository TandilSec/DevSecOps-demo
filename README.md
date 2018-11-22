# Introduccion

## Charla de Seguridad en optativa DevOps UNICEN 

<!-- <iframe src="https://docs.google.com/presentation/d/e/2PACX-1vSZZKrs1fqfvyo24vhiQvjtJw_QmV1PFcK1wMBd8ClEDaKW4EYuirDUr7ux4fDV98TkCKAAQ_ZiLkqe/embed?start=false&loop=false&delayms=3000" frameborder="0" width="1280" height="749" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe> -->


# Aplicacion base

    mkvirtualenv --python=python3.6 djangoapp
    pip install --upgrade pip setuptools wheel Django==2.1.3

    django-admin startproject djangoapp

    nano requirements.txt

      Django==2.1.3
      gunicorn==19.9.0

    nano Dockerfile

      FROM python:3.6-alpine

      ADD . /usr/src/app
      WORKDIR /usr/src/app

      COPY requirements.txt ./

      RUN pip install --no-cache-dir -r requirements.txt

      EXPOSE 8000

      CMD exec gunicorn djangoapp.wsgi:application --bind 0.0.0.0:8000 --workers 3

    docker build -t django_app_image .

    docker run --name django_app -p 8000:8000 -i -t django_app_image

# Chequeo de dependencias

    docker pull owasp/dependency-check
    docker run -i --volume dependency-check-data:/usr/share/dependency-check/data --volume "$(pwd)":/src owasp/dependency-check --scan /src --project "Test"

# Analisis dinamico de codigo

    docker pull owasp/zap2docker-stable
    docker run -i owasp/zap2docker-stable zap-baseline.py -t http://172.17.0.2:8000/
    docker run -i owasp/zap2docker-stable zap-cli quick-scan --self-contained --start-options '-config api.disablekey=true' http://172.17.0.2:8000/

# Chequeo de seguridad de contenedor

## Instalar clair-scanner

    go get -u github.com/golang/dep/cmd/dep
    git clone https://github.com/arminc/clair-scanner.git src/clair-scanner/
    cd src/clair-scanner/ && make ensure && make build

## Lanzar una instancia de Clair y su base de datos

    make db
    make clair

## Analizar la imagen con Clair

    ./clair-scanner --ip=172.17.0.1 django_app_image

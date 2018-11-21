


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

## Instalar analyze-local-images

  git clone https://github.com/coreos/analyze-local-images $HOME/analyze-local-images-gopath/src/github.com/coreos/analyze-local-images
  export GOPATH=$HOME/analyze-local-images-gopath
  cd $HOME/analyze-local-images-gopath/src/github.com/coreos/analyze-local-images
  glide install
  go install github.com/coreos/analyze-local-images

## Instalar clair-scanner

  git clone https://github.com/coreos/analyze-local-images $HOME/analyze-local-images-gopath/src/github.com/coreos/analyze-local-images
  export GOPATH=$HOME/analyze-local-images-gopath
  cd $HOME/analyze-local-images-gopath/src/github.com/coreos/analyze-local-images
  glide install
  go install github.com/coreos/analyze-local-images

## Lanzar una instancia de clair y su base de datos

  docker run -p 5432:5432 -d --name clair-db arminc/clair-db
  docker run -p 6060:6060 --link clair-db:postgres -d --name clair arminc/clair-local-scan:v2.0.6

## Analizar la imagen

  ./analyze-local-images -endpoint http://172.17.0.3:6060 -my-address 172.17.0.3 django_app_image

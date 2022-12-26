# Documentación creación de un repo helm
https://tech.paulcz.net/blog/creating-a-helm-chart-monorepo-part-1/

Helm es una buena herramienta para empaquetar, compartir y desplegar Manifest de Kubernetes.
Podemos compartir Charts (packages) vía repositorios Helm. Se trata básicamente de websites estáticos con un index.html, que provee metadatos y links sobre los packages Helm.

Estos sites pueden estar ubicados en github, en s3, etc...

Para nuestro caso de uso utilizaremos un repositorio github (https://github.com/gincol/helm-charts.git)

## Preparación local
Creamos dicho repositorio en nuestra cuenta github y lo clonamos (aún vacío) a nuestro ordenador de trabajo.

git clone https://github.com/gincol/helm-charts.git
cd helm-charts

A continuación crearemos un directorio que contendrá todos nuestros packages

mkdir charts
cd charts

Creamos la estructura de nuestro primer chart, destinado a una app springboot
helm create springboot

Subimos cambios
git add .
git commit -m "Initial Commit"
git push -u origin main

## Preparación github pages
Creamos una rama llamada gh-pages
git checkout -b gh-pages
git push origin gh-pages

Revisamos si tenemos habilitado github pages en nuestro repo

Para construir y subir todo lo necesario podemos usar comandos helm package y helm repo, o bien usar chart-releaser.
Lo instalamos en nuestro mac según https://github.com/helm/chart-releaser
brew tap helm/tap
brew install chart-releaser

cr help

Usaremos dos comandos
cr index, el cual crea el fichero index.html
cr upload, que sube los packages a github releases

Para este segundo necesitaremos crear un token developer.
Una vez creado y anotado, creamos el fichero ~/.cr/cr.yaml con este contenido
owner: XXXXX
git-repo: helm-charts
package-path: .deploy
token: XXXXXXXXXX
git-base-url: https://api.github.com/
git-upload-url: https://uploads.github.com/

## Empaquetamos y subimos
helm package charts/springboot --destination .deploy
cr upload


# Creación de un repo Helm


# Install repo
helm repo add k8sObs-as-helm https://github.com/gincol/helm-charts.git

# Install chart
helm install -f dev/values.yaml springboot-obs ../springboot-helm-obs

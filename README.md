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

## Creamos el index
cr index -i ./index.yaml

queda con este contenido:

apiVersion: v1
entries:
  springboot:
  - apiVersion: v2
    appVersion: 1.16.0
    created: "2022-12-26T18:07:27.337882+01:00"
    description: A Helm chart for Kubernetes
    digest: 6a23ff6695a106db8ae748f43bdc5d654cc1ba871ca4e0a28fb94029c69781f6
    name: springboot
    type: application
    urls:
    - https://github.com/gincol/helm-charts/releases/download/springboot-0.1.0/springboot-0.1.0.tgz
    version: 0.1.0
generated: "2022-12-26T18:07:27.337893+01:00"

Lo subimos al repo
git add .
git commit -m "release 0.1.0"
git push origin gh-pages

# Testeamos
helm repo add gincol-charts https://gincol.github.io/helm-charts/
helm install -f values.yaml springboot-normal --namespace dev gincol-charts/springboot

En caso de necesitar upgrade:
helm upgrade springboot-normal gincol-charts/springboot -f values.yaml -n dev

Viendo el histórico de despliegues 
helm history springboot-normal -n dev

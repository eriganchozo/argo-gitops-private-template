# GitOps wordpress demo

Este repositorio, nos mostrará como podremos crear un despliegue basado en
GitOps usando el [chart público de wordpress de bitnami](https://github.com/bitnami/charts/tree/main/bitnami/wordpress).

Como puede observarse desde la documentación del chart, es posible desplegar la
base de datos con el propio wordpress o conectar con una base de datos externa.
Nuestro flujo, promueve el desacoplamiento de la base respecto del chart porque
es un antipatrón que un chart tenga como dependencia una base de datos para
aquellos escenarios donde se corren migraciones que deberían ejecutarse en un
helm hook de pre-install,pre-upgrade.

Por esta razón, este repositorio lo separamos en dos charts, cada uno en una
subcarpeta de la actual:

* **`reqs/`:** sería el chart que desplegaría el MariaDB. No será más que un
  chart que depende del chart que definitivamente instale MariaDB. Ya veremos
  cómo proceder con este ejemplo.
* **`wp/`:** sería el chart que desplegaría el Wordpress. No será más que un
  chart que depende del chart que definitivamente instale Wordpress.

La idea de ambos charts, es la de ser dependientes de los charts que
realmente generen templates. Aquí únicamente tendremos los valores versionados,
y como seguramente tengamos datos sensibles, los cifremos para el ambiente que
éste repositorio representará su despliegue.

> Esto significa que **ninguno de estos charts creará templates**. Los templates
> **los crean aquellos chart desde los que se depende**.

## Requisitos

Como se mencionó en el [readme general](../) se recomienda utilizar [direnv](https://direnv.net/)
para el seteo de variables al ingresar al directorio. Si bien no es necesario,
debe tenerse presente a la hora de cifrar los datos, estén seteadas las variables
`SOPS_AGE_RECIPIENTS` y `SOPS_AGE_KEY_FILE` para poder trabajar con
[sops](https://github.com/mozilla/sops). Otra cosa que se mencionó en el readme
general es la importancia de contar con la clave pública Age usada por Argo CD.
De esta forma, procedemos a configurar:

* Una clave Age nueva para cifrar los datos para el wordpress
* La clave Age de Argo CD seteada en el directorio anterior será necesaria para
  configurar `SOPS_AGE_RECIPIENTS`

Entonces, generamos el `.envrc` de la siguiente forma:

```bash
# Seteamos SOPS_AGE_KEY_FILE auxiliarmente para indicar el PATH a nuestra clave
# Age privada
SOPS_AGE_KEY_FILE=$PWD/.age/key

# Creamos el directorio para alojar la clave Age
mkdir -p .age

# Creamos la clave privada propia de éste repositorio
age-keygen -o $SOPS_AGE_KEY_FILE

# Obtenemos la clave pública de la reciente clave generada
AGE_PK=$(cat $SOPS_AGE_KEY_FILE| grep 'public' | cut -d: -f2 | tr -d ' ')

# Creamos el .envrc con los datos recientemente obtenidos
cat > .envrc <<EOF
source_up
export SOPS_AGE_KEY_FILE=\$PWD/.age/key
export SOPS_AGE_RECIPIENTS=${AGE_PK},\${ARGOCD_AGE_PK}
EOF
```

> Observar que la variable `ARGOCD_AGE_PK` se obtiene con la instrucción
> `source_up`, cargando las variables seteadas por un `.envrc` en alguna carpeta
> padre de la actual.

Escrito el nuevo `.envrc` direnv nos pedirá permitir cargar sus variables:

```bash
direnv allow
```

## Cifrando los valores de cada despliegue

Hemos brindado en este repositorio los datos listos para configurar un
despliegue de un wordpress que se servirá en la URL
http://wp-example-prod.gitops.localhost. La contraseña será
**mikroways-production!**. 

Estos datos tenemos que cifrarlos. Para ello:

```bash
sops -e reqs/secrets.clear.yaml > reqs/secrets.yaml
sops -e wp/secrets.clear.yaml > wp/secrets.yaml
```

> Los archivos `secrets.clear.yaml` no deberían versionarse. Aquí se versionan
> para simplificar el ejemplo y el cifrado de los datos guiado por este
> documento.

Una vez finalizados todos los cambios, subirlos a git y hacer push:

```bash
git add .
git commit -m "Encrypt secret data"
git push origin main
```

Cuando este directorio tenga los datos subidos, podrá continuarse con el
[ejemplo de
GitOps](https://github.com/Mikroways/argo-gitops-demo-example/tree/main/projects?#un-ambiente-con-un-despliegue-usando-un-repositorio-externo-de-gitops).

## Qué ignora este repositorio

El repositorio actual, ignora:

* La clave Age para así evitar versionar una clave
* El archivo `Chart.lock` para permitir a Argo CD detectar cambios cuando el
  chart del que depende nuestro chart saca un nuevo **patch number**. Esto es
  posible porque el `Chart.yaml` define como dependencia un **asterisco** en la
  versión del **patch number**. Si se versionara el `Chart.lock` esto no sería
  posible, porque Argo CD respetaría lo que el **indique el lock*.
* El directorio `charts/` que contiene las dependencias si corremos `helm
  dependency update`

## Cómo crear los charts en este repositorio

Los directorios `req` y `wp`, se han creado para simplificar la labor de quien
esté probando el flujo. Pero si se quisieran crear, simplemente se deben correr
los siguientes comandos:

```bash
helm create wp
helm create reqs
rm -fr {wp,reqs}/{charts,templates,values.yaml} 
```

Luego, hemos editado los único archivos dentro de cada directorio, es decir 
`wp/Chart.yaml` y `reqs/Chart.yaml`. Cada uno agrega como dependencia el o los
charts de quien dependen.

## Cómo probar los templates que generaría este repositorio

Suponiendo que se desea visualizar los templates de **req/**:

```bash
helm dependency update reqs/
helm secrets template reqs/ -f reqs/secrets.yaml
```

Para el caso de **wp/**_

```bash
helm dependency update wp/
helm secrets template wp/ -f wp/secrets.yaml
```

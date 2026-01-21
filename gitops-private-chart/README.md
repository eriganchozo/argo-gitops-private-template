# GitOps nginx con un chart privado

La idea de este repositorio, es la de desplegar en ArgoCD una aplicación que
dependerá de un chart que está subido a un repositorio privado de Helm Charts.

La explicación de cómo crear y subir un chart a un repositorio privado, puede
seguirse leyendo [la documentación provista en `../charts`](../charts).

Considerando que ya has generado el paquete con el chart en una registry
privada como se explica, la idea es modificar el archivo aquí presente
`Chart.tpl.yaml` **cambiando la URL del repositorio del chart** por la que
corresponda con tu repositorio. Es importante tener presente el usuario y
contraseña. Todos estos datos se obtienen como se explica cómo trabajar con
[repositorios de helm charts](../charts#helm-repositories).

## Requisitos

A diferencia de los otros ejemplos, en este caso usaremos un chart sin datos
sensibles. Por tanto, aquí no nos preocuparemos por cifrar datos.

## Modificar `values.yaml`

Simplemente, es necesario modificar el archivo `values.yaml` que puede copiarse
y parametrizar sus valores a partir de `values.tpl.yaml`. La idea es simplemente
completar los campos que dicen CHANGEME.

Una vez finalizados todos los cambios, subirlos a git y hacer push:

```
git add .
git commit -m "Add custom values.yaml"
git push origin main
```

El nginx personalizado, podrá accederse con el ingress aquí configurado que
tendrá la url http://mikroways-website.gitops.localhost.

Una vez editados los valores y subidos a git, podrá continuar con el [ejemplo de
GitOps que utiliza un chart almacenado en un repositorio privado](https://github.com/Mikroways/argo-gitops-demo-example/tree/main/projects?#un-ambiente-con-un-repositorio-externo-de-gitops-que-utiliza-un-chart-privado).

> Debe prestar especial atención al ejemplo basado en un repositorio de charts y
> **no el que utiliza registries OCI**.

## Qué ignora este repositorio

El repositorio actual, ignora:

* El archivo `Chart.lock` para permitir a Argo CD detectar cambios cuando el
  chart del que depende nuestro chart saca un nuevo **patch number**. Esto es
  posible porque el `Chart.yaml` define como dependencia un **asterisco** en la
  versión del **patch number**. Si se versionara el `Chart.lock` esto no sería
  posible, porque Argo CD respetaría lo que el *indique el lock*.
* El directorio `charts/` que contiene las dependencias si corremos `helm
  dependency update`

## Cómo crear este directorio

El directorio actual fue creado usando, desde la carpeta inmediatamente
exterior, el siguiente comando:

```
helm create gitops-custom-nginx
cd gitops-custom-nginx
rm -fr {charts,templates,values.yaml} 
```

Luego, hemos editado `Chart.yaml` agregando como dependencia el chart de quién
dependen.

# GitOps nginx con un chart con dependencia de otro usando registries OCI

No tiene sentido diferenciar este ejemplo, del que usa un [repositorio privado
de helm](../gitops-private-chart). Por ello, simplemente diremos que aplica la
misma documentación, sólo que cambiamos la dependencia usando una url
**oci://** en vez de **http://**. Por ello, prestar especial atención al
archivo `Chart.tpl.yaml` utilizando la url como se explica en [la documentación
específica del uso de registries OCI para helm charts](../charts#helm-registries).

Este ejemplo, nuevamente desplegará la web de [Mikroways](https://www.mikroways.net)
usando un  nginx personalizado. El despliegue podrá accederse con el ingress
configurado cuya url es http://mikroways-website-oci.gitops.localhost.

Una vez editados los valores y subidos a git, podrá continuar con el [ejemplo de
GitOps que utiliza un chart almacenado en un repositorio privado](https://github.com/Mikroways/argo-gitops-demo-example/tree/main/projects?#un-ambiente-con-un-repositorio-externo-de-gitops-que-utiliza-un-chart-privado).

> Debe prestar especial atención al ejemplo basado en registries OCI y **no al
> de repositorios tradicionales helm**.

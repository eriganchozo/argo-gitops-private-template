# Ejemplos varios desplegado con GitOps

Este repositorio, mantiene varios ejemplos de GitOps que son referenciados desde
el [flujo propuesto por Mikroways](https://github.com/Mikroways/argo-gitops-demo-example/).

Este repositorio en un ejemplo real, no debe ser un único repositorio con varios
directorios, sino que cada directorio debería existir en un repositorio
independiente. Obviamente que esto dependenrá de cada usuario del flujo, pero
mantener repositorios independientes ofrece mayor desacoplamiento y los permisos
sobre cada uno se vuelve más granular. Así como el esquema de trabajo del equipo
que mantenga ese despliegue GitOps.

## Ejemplos en este repositorio

Debido a que el flujo presenta varios ejemplos, con este repositorio
completaremos los ejemplos que requieren trabajar con:

* [**Un repositorio de GitOps**](gitops-wordpress/): desplegará un wordpess
* [**Un repositorio de GitOps con una registry privada**](gitops-custom-nginx/):
  desplegará un nginx desde una imagen privada, usando un chart público.
* **Un repositorio de GitOps utilizando un repositorio de chart privado:**
  dividimos este ejemplo en dos escenarios. Ambos harán exactamente el mismo
  despliegue, pero uno usará un repositorio de charts privado estándar mientras
  que el otro usará una registry OCI como repositorio privado de helm charts:
  * [**Ejemplo con un chart en un repositorio de charts privado**](./gitops-private-chart)
  * [**Ejemplo con un chart en una registry OCI privada**](./gitops-private-chart-oci)

## Uso del repositorio

Este repositorio, no deberá usarse directamente, sino que la idea, es crear un
nuevo repositorio **personal y privado** desde el botón [template](https://github.com/Mikroways/argo-gitops-private-template/generate).

Una vez creado el repositorio privado seguir la documentación de cada
directorio y del flujo para así completar la experiencia.

> Es recomendable probar crearlo en github y en gitlab para verificar las
> experiencias con diferentes plataformas. Este repositorio incluye pipelines
> para generar imágenes de contenedores en ambas plataformas al pushear a la
> rama principal: **main**.

## Requerimientos

Para el uso de GitOps, manejaremos claves Age que nos permitirá cifrar los datos
usando sops. Las instrucciones para instalar las herramientas se mencionan en el
[repositorio del flujo de gitops](https://github.com/Mikroways/argo-gitops-demo-example/tree/main/kind#requerimientos).
Para este repositorio, no necesitamos todas las herramientas allí mencionadas,
solo necesitamos de:

* [age](https://age-encryption.org/)
* [sops](https://github.com/mozilla/sops)
* [direnv](https://direnv.net/)

### Configuramos entonces direnv

Usaremos como configuración global en este repositorio, una variable llamada
`ARGOCD_AGE_PK` que luego, cada ejemplo utilizará para cifrar datos con una
clave propia y la pública de Argo CD almacenada en la variable mencionada.

Creamos entones esta variable de la sigiente forma: desde un directorio donde
tengamos acceso al cluster kind creado como se explica en [la documentación del
ejemplo del marco de
trabajo](https://github.com/Mikroways/argo-gitops-demo-example/tree/main/kind#obtener-la-clave-age-p%C3%BAblica-de-argo-cd)
corremos el siguiente comando:

```bash
# Accedemos al directorio donde esté corriendo el cluster kind y tengamos un
# kubeconfig que nos permita correr el siguiente comando
#     cd <directorio-con-acceso-a-cluster-kind>
AGE_PK=$(kubectl -n argocd get secrets -l component=helm-secrets-age \
  -o jsonpath='{.items[0].data.key\.txt}' | base64 -d \
  | grep 'public' | cut -d: -f2 | sed 's/^ //')

# Una vez con la variable seteada, procedemos a retornar al directorio con el
# repositorio clonado desde el template
#     cd <vuelvo al directorio de este repositorio>
cat > .envrc <<EOF
export ARGOCD_AGE_PK=$AGE_PK
EOF
```

Al ejecutar el comando anterior, **direnv nos solicitará permitir el `.envrc`
generado**:

```bash
direnv allow
```


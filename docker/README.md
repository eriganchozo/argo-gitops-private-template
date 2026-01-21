# Custom nginx docker image

Este directorio provee una imagen de un contenedor correspondiente a un nginx
personalizado: la imagen servirá la web de [Mikroways](https://www.mikroways.net/).
Puede observarse que se proveen pipelines para:

* **GitHub Actions:** pushea una imagen en ghcr.io. El pipeline está definido en [la
  carpeta ../.github](../.github).
* **GitLab CI:** puseha una imagen en registy.gitlab.com. El pipelines está definido
  en [../.gitlab-ci.yml](../.gitlab-ci.yml).

> Ambos pipelines corren cuando se pushea en la rama principal:
> **main**.

## Configurar el acceso a la registry privada de GitHub

En GitHub, el pipeline correrá y creará una imagen en
`ghcr.io/username/project-name/nginx:latest`. Una vez que se suba la imagen,
será posible verificar su visibilidad como se muestra en las siguientes
imágenes:

![gh registry](./assets/gh-registry-access.png)

> Haciendo click en 1, podremos ver un detalle de nuestros paquetes, que en este
> caso son imágenes.

![gh registry details](./assets/gh-registry-access-details.png)

> Como se muestra en 1, el nombre de la imagen es el mencionado anteriormente:
> ghcrc.io, seguido del username, seguido del nombre del repositorio y
> finalmente nginx. Siempre se generará el tag latest. En 2, podemos verificar
> que el acceso es privado.

![gh registry visibility](./assets/gh-registry-access-visibility.png)

> En Danger Zone podemos verigicar que se trata de un acceso privado, teniendo
> entonces que configurar un GitHub Personal Access Token.

Es importante verificar que no podemos hacer un `docker pull` de dicha imagen.
Por supuesto que asumimos que no se encuentra logeuado en **ghcr.io**.

> Puede verificar el comando `docker pull` luego de haber realizado un `docker
> logout ghcr.io`

### Creamos un GitHub Personal Acecss Token

Accedemos a [Settings, Developer Settings, Tokens (classic)](https://github.com/settings/tokens) y **generaremos uno
nuevo (classic)**:

![gh registry access token classic](./assets/gh-registry-access-token-classic.png)

> Observamos que en 1 seleccionamos que no expire, mientras que en 2 que este
> token únicamente pueda descargar paquetes (en nuestro caso imágenes).

El resultado, será una token con un formato parecido a
`ghp_mXkUqp4Kt88BPGD1kvWz372y7JyRZ62OJ9s8`. Copiarlo y reservarlo para luego
utilizar este token como clave, y username el propio de github, para poder
probar con docker:

```
# Por las dudas primero nos deslogeuamos:
docker logout ghcr.io

# Ahora nos loguearemos con el nuevo token:
docker login ghcr.io -u <gh-username>
```
> La contraseña será el token obtenido.

Probar ahora si es posible realizar un `docker pull` de la imagen creada.


## Configurar el acceso a la registry privada de GitLab

Gitlab también generará una imagen en la registry de gitlab siguiendo el nombre
`registry.gitlab.com/username/project-nam/nginx:latest`. En este caso, Gitlab
mantendrá la registry privada si el repositorio es privado. Verificaremos si
podemos descargar la imagen:

```bash
# Aseguramos estar sin sesión para gitlab
docker logout registry.gitlab.com
```

Hacemos un `docker pull` para verficar que no es posible realizar la descarga.

![gitlab image name](./assets/gl-registry-image-name.png)

> En la captura mostramos cómo obtener el nombre de nuestra imagen accediendo en
> 1, y luego clickeando en 2.


Continuamos luego con la configuración de un token de acceso a la registry:

![gitlab access token](./assets/gl-registry-access-token.png)o

> Haciendo clic en Settins, Repository (1), accedemos a varias opciones.
> Expandimos Deploy tokens (2) y completamos los datos: nombre del token (3),
> una expiración del token que puede ser opcional (4), un usuario (5), permisos
> para leer de la registry  (6) y damos click en 7. La contraseña será algo como
> `ekzc4PB9gLV1WBB1QA4Y`

Verificamos que podemos hacer pull:

```bash
# Nos loguearemos con el nuevo token (observar que ahora el username puede ser
# otro diferente al usado por nuestro usuario en gitlab:

docker login registry.gitlab.com -u registry
```

> La contraseña será el token obtenido.

Probar luego si podemos hacer un pull de la imagen.

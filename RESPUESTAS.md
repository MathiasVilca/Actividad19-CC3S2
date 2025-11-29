## Bloque 1
**¿Por qué microservicios?**
Al inicio, los sistemas eran monolíticos (es decir, todos los servicios/módulos estan embebidos en una sola app), pero debido a ser más fragiles a fallos y al ser difíciles de mantener y escalar, se paso a SOA. SOA dividió ligeramente los servicios, minimizando dependencias (Acoplamiento débil), pero todavía necesitaban comunicación y compartian fuertemente ciertos recursos.  Finalmente, se logro separar los servicios en microservicios, los cuales son independientes en despliegue y datos.  
Ejemplos:
- Un E-Commerce el cual tiene picos estacionales: El escalamiento de una app monolítica requiere duplicación completa, lo cual es costoso e ineficiente.
- Un SaaS multi-inquilino: si un error ocurre y uno de los módulos falla, entonces toda la app se detiene, afectando a todos los clientes sin importar que servicio hayan estado usando.

**Críticas al monolito**
- **Cadencia de despliegue reducida:** Un solo cambio en un módulo específico requiere el redespliegue de toda la aplicación, el cual tiene riesgo de romper algo en otro módulo.
- **Acoplamiento:** Si se requiere que un modulo use más recursos, se tiene que duplicar también el resto de módulos (que probablemente no necesiten tantos recursos), desperdiciando infraestructura.

**Popularidad y beneficios**
Se adoptaron justamente porque la independencia de microservicios: 
- Aisla los fallos del resto, permitiendo que estos sigan en funcionamiento.
- Permite asignar más o menos recursos a aquellos servicios que los necesitaran o sobraran.
- Permiten el trabajo de distintos equipos a la vez, sin que uno se tropezara con el otro a la hora de hacer cambios a pesar de estar trabajando en módulos distintos.

**Desventajas y retos**
Desafíos:
- La comunicacion entre servicios se hace por red, lo cual puede fallar o ser lento o inseguro.
- El orquestamiento de todos los microservicios puede ser complejo.
- Es más dificil mantener consistencia de datos de módulos diferentes
- El probar errores de interacción entre servicios puede ser complicado.  

Mitigaciones:
- OpenAPI y contratos: Mantienen consistencia a la hora de la comunicación entre servicios.
- Patrones de sagas: Secuencias de eventos las cuales deshacen cambios en caso fallase la actualizacion de datos en alguno de los servicios.
- Trazabilidad (Jaeger): Permite ver que camino toma una petición al llegar a la app y ver en que parte falla.
- Para orquestación, uso de herramientos de automatización como kubernetes.

**Principios de diseño**
- **DDD (Diseño dirigido por dominios):** Al usar limites contextuales, se define hasta donde llega un microservicio y empieza otro, como dividirlos por su funcionalidad en la app (Envíos, Facturación, ..).
- **DRY (Don't Repeat Yourself):** DRY busca no repetir código, pero el compartir librerías entre servicios puede generar acoplamiento, asi que se prefiere tener duplicacion controlada.
- **Criterios de tamaño:** Tamaño de un servicio se mide en resposabilidad, no en cuantas tablas o lineas de codigo tiene. Debe encapsular una funcionalidad completa y contener todo lo estrictamente necesario para cumplirla.

## Bloque 2
**Porque no usar `:latest`?**
- `:latest` es bastante ambiguo ya que no distingue que version es, hoy puede ser la `1.1.0` y mañana la `2.0`, cosa que puede malograr el entorno en caso este no se compatible con la nueva version
- SemVer (o Versionado Semántico) soluciona esto al dar un identificador único a cada versión, dependiendo de la escala de los cambios hechos respecto a la otra (primer número se cambia si es un cambio mayor, el segundo si es menor, y el tercero si es un parche). Permite reproducibilidad al fijar una version con la cual se sabe el entorno funciona

## Bloque 3
### Docker compose para desarollo 
**Teórico:**  
**Ventajas de docker compose:**  
- Declaratividad: Con `docker run` se deben ejecutar multiples comandos en orden para alcanzar la arquitectura deseada, mientras que con docker compose se describe esta en un solo archivo.
- Gestión automatica de redes: Mientras que `docker run` requiere creacion manual de redes y conexiones con contenedores, compose crea una red por defecto para la comunicación de su servicios, tambien se puede definir por servicio via `networks:`.
- Dependencias entre servicios: Permite ordenar arranque al usar `depends_on`
- Soporte para perfiles: Permite "agrupar" ciertos servicios en perfiles usando `profiles: `, los cuales permiten levantar servicios específicos a la hora de arrancar.
- Entornos reproducibles: Toda la informacion se guarda en un archivo `docker-compose.yml`, el cual es versionable en Git y permite reproducir el mismo entorno donde sea con `docker compose up`

**Ejercicios:**
1. Tres escenarios conde compose mejora flujo:
 - **Staging local:** Se puede replicar exactamente la arquitectura de producción usando `docker compose up`, sin necesidad de comandos adicionales.
 - **Pruebas de integración:** Facilita orquestación de dependencias reales, ya que puede levantar y tener listos los contenedores de BDs o de un servicio de caché si se requieren para este tipo de pruebas.
 - **Recarga en vivo (Hot-Reload):** Se puede vincular una carpeta de nuestra computadora con una en el contenedor, permitiendo que nuestros cambios se reflejen en el contenedor al instante. Por ejemplo, si se tiene un bind mount `./app:/app`, nos permite cambiar nuestro codigo de app desde local y que `uvicorn --reload` detecte estos cambios y reinicie la app.

2. Por qué usar perfiles:
 - El usar perfiles permite separar servicios por entornos, por ejemplo, el entorno `dev` con bind mounts para edición en vivo es diferente del entorno `test`, el cual no tiene estos ya que se busca probar.
 - Tambien permite no ejecutar servicios innecesarios, por ejemplo, herramientas de monitoreo a la hora de hacer pruebas.

3. Fragmento conceptual de `docker-compose.yml`
```yml
version: '3.8'

services:
  api:
    image: ejemplo-api-service:0.2.0 #nombre de imagen que se va a construir
    build: . # busca dockerfile en carpeta actual
    ports:
      - "8080:80" #conecta puerto 8080 a puerto interno 80
    volumes:
      - ./app:/app #bind mount: conecta ./app a la carpeta del contenedor /app
    depends_on:
      - cache # ordena arranque: espera a cache
    command: uvicorn main:app --reload --host 0.0.0.0 --port 80 #sobreescribe CMD de dockerfile, --reload garantiza que al hacerse un cambio en los archivos, se recarga la app
  
  cache:
    image: redis:alpine #agarra imagen oficial de redis, no necesita especificar build
    ports:
      - "6379" #puerto interno, solo expone puerto
```
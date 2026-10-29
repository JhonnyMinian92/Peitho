# Desarrollo local con Docker y PostgreSQL instalado en el host

## Estado de esta guía

Esta guía define la configuración objetivo. En el estado documental actual todavía deben crearse el frontend, backend, Dockerfiles y compose.yaml; por eso los comandos de arranque solo se consideran operativos cuando las tareas PEI-011 a PEI-029 estén cerradas y se hayan probado en una instalación limpia.

## Decisión obligatoria

- Las aplicaciones, workers y servicios de IA se construyen y ejecutan en contenedores Docker durante el desarrollo local.
- PostgreSQL de desarrollo se instala y ejecuta localmente en el sistema anfitrión, fuera de Docker Compose.
- La instancia local debe tener la versión fijada por ADR y la extensión pgvector instalada y habilitada en la base de Peitho.
- compose.yaml no debe definir un servicio postgres ni publicar un puerto de base de datos.
- Los contenedores se conectan al host mediante host.docker.internal. En Docker Engine sobre Linux se añade host-gateway si el nombre no está disponible de forma automática.
- No usar una base de producción ni credenciales o datos reales para pruebas locales.

## Componentes objetivo

| Componente | Forma de ejecución en local | Responsabilidad |
|---|---|---|
| Angular | Contenedor Docker | UI, build y servidor de desarrollo o runtime local |
| FastAPI | Contenedor Docker | API, contratos JSON, auth, CRM y dominios |
| Worker/scheduler | Contenedor Docker separado o proceso con comando dedicado | Procesos asíncronos y tareas programadas |
| Embeddings | Contenedor Docker Python | Servicio Sentence Transformers local |
| LLM de desarrollo | Perfil Docker opcional con Ollama | Pruebas locales de generación |
| vLLM | Perfil opcional, solo si hardware y drivers son compatibles | Runtime previsto para inferencia de producción |
| PostgreSQL + pgvector | Servicio instalado en el host | Base local de Peitho; no se arranca mediante Compose |
| Inventary | Entorno autorizado separado | Solo réplica/lectura; no se utiliza producción desde tests |

## Requisitos previos

- Docker Desktop o Docker Engine con Compose Plugin, correctamente iniciado.
- Node.js y gestor de paquetes solo si se desea ejecutar tooling Angular directamente en el host; la ejecución normal debe ser reproducible desde la imagen.
- Python solo para tooling local opcional; la imagen del backend fija el runtime oficial.
- PostgreSQL en el host, cliente psql disponible, extensión pgvector instalada en la instancia.
- Recursos de RAM/disco suficientes para builds, workers y modelos elegidos. El modelo LLM no se descarga/arranca por defecto sin seleccionar y documentar uno compatible.
- Git con los archivos del repositorio actualizados.

## Variables de entorno

Crear un archivo .env local a partir de .env.example y no subirlo a Git. El archivo de ejemplo solo contiene nombres y valores ficticios. Las variables de conexión deben incluir al menos:

| Variable | Valor local objetivo | Nota |
|---|---|---|
| PEITHO_DATABASE_HOST | host.docker.internal | Dentro de contenedor; desde proceso host se usa localhost |
| PEITHO_DATABASE_PORT | 5432 o puerto local configurado | No publicar ni mapear el puerto desde Compose |
| PEITHO_DATABASE_NAME | peitho_dev | Base exclusiva de desarrollo |
| PEITHO_DATABASE_USER | peitho_app | Rol de runtime con mínimo privilegio |
| PEITHO_DATABASE_PASSWORD | definida en .env local | Nunca usar valor real en documentación, código o logs |
| PEITHO_DATABASE_SSLMODE | verify-full/require según certificados y configuración local | No desactivar verificación silenciosamente en ambientes compartidos |
| PEITHO_ENVIRONMENT | local | Selección explícita del ambiente |
| PEITHO_LOG_LEVEL | INFO | Sin datos sensibles |

Los nombres finales de variables deben mantenerse sincronizados con settings del backend y .env.example. No almacenar claves privadas, tokens de agentes ni secretos de cifrado en archivos versionados.

## Preparar PostgreSQL local

1. Instalar la versión aprobada en el ADR y confirmar que el servicio está iniciado.
2. Confirmar conectividad local con psql y documentar comandos equivalentes para Windows, macOS y Linux.
3. Instalar pgvector de forma compatible con la distribución elegida. Crear la extensión en peitho_dev mediante la migración o rol autorizado.
4. Crear roles separados: owner/migrator para aplicar migraciones, runtime para la aplicación, y roles de lectura específicos cuando una integración lo requiera. No ejecutar el backend como superusuario.
5. Configurar listen_addresses solo sobre la interfaz local y/o interfaz de gateway necesaria. Si el contenedor necesita conectar a una IP del host, permitirla explícitamente; no abrir PostgreSQL a toda la red pública.
6. Configurar pg_hba.conf con scram-sha-256 y la dirección/subred estrictamente necesaria de Docker. No usar reglas abiertas a cualquier dirección como solución permanente.
7. Restringir el firewall a la red de Docker local. Verificar conexión fallida desde redes no autorizadas.
8. Crear peitho_dev vacía y aplicar migraciones versionadas. Cargar únicamente datos sintéticos.

La configuración de la interfaz del host depende del sistema operativo y de la instalación de PostgreSQL. El agente debe documentar la configuración exacta validada en la máquina objetivo; no copiar a ciegas listen_addresses='*' ni abrir 5432 a Internet.

## Configuración de Compose

El compose.yaml final debe cumplir:

- No contiene un servicio llamado postgres ni imagen PostgreSQL.
- No mapea el puerto 5432 porque PostgreSQL está en el host.
- Todos los servicios consumen la configuración por variables de entorno.
- Los secretos de ejecución vienen de .env local ignorado por Git o del gestor de secretos del ambiente.
- En Linux, los servicios que lo requieran declaran host.docker.internal:host-gateway.
- El frontend y API tienen healthchecks, límites razonables, redes explícitas y logs consultables.
- El worker y scheduler no duplican tareas; su ejecución es idempotente y se puede detener con elegancia.
- Los modelos pesados usan perfiles optativos y no impiden levantar los módulos básicos de aplicación.
- Las imágenes se construyen desde el código del repositorio, usan dependencias fijadas y ejecutan como usuario no root.
- Se usa una red interna para dependencias no públicas; solo se publican puertos de UI/API necesarios para el host.

## Flujo normal de trabajo

Una vez implementados los servicios, desde la raíz del repositorio:

~~~bash
docker compose config
docker compose build
docker compose up -d
docker compose ps
docker compose logs -f
~~~

Para comprobar el entorno, el agente debe verificar en orden: contenedores saludables, API lista, conexión a peitho_dev, migraciones aplicadas, extensión vector disponible y frontend capaz de consultar la API.

Para detener los servicios de aplicación:

~~~bash
docker compose down
~~~

No utilizar docker compose down -v como comando rutinario. Aunque el Compose no debe contener PostgreSQL, eliminar volúmenes puede destruir otros datos de desarrollo asociados a servicios de la aplicación. Nunca borrar la base local con fines de limpieza sin copia y confirmación expresa.

## Diagnóstico

- Contenedor no conecta a PostgreSQL: comprobar que el servicio local está activo, dirección gateway, DNS host.docker.internal, firewall, listen_addresses, pg_hba.conf, puerto y credenciales.
- localhost no funciona desde un contenedor: localhost apunta al propio contenedor, no al host; usar host.docker.internal.
- pgvector no está disponible: verificar la instalación de la extensión en el servidor local compatible, no solo en el cliente psql.
- Compose no conoce un host en Linux: declarar host-gateway y revisar la ruta al gateway Docker.
- Falla la autenticación: rotar/actualizar .env local, no pegar secretos en issues, logs o chat.
- Los datos están obsoletos: consultar estado de watermark, replicación y conciliación; no desactivar las guardas de negocio.

## Criterios de aceptación del entorno local

- [ ] compose.yaml no arranca PostgreSQL ni publica su puerto.
- [ ] El backend Docker se conecta al PostgreSQL local del host con rol de mínimo privilegio.
- [ ] Las migraciones funcionan contra una base vacía y habilitan las extensiones requeridas.
- [ ] Las pruebas y seeds usan datos sintéticos.
- [ ] Los servicios tienen healthchecks/logs y pueden reconstruirse desde Dockerfiles reproducibles.
- [ ] Un desarrollador nuevo consigue arrancar, probar y apagar el entorno siguiendo solo esta guía y el README.
- [ ] El firewall y pg_hba.conf impiden accesos no autorizados desde fuera del entorno local.

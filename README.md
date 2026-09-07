### Pregunta 1: ¿Por qué no se recomienda desarrollar directamente sobre `main` en este laboratorio?

No se recomienda trabajar directamente sobre la rama `main` por varios motivos prácticos de calidad, orden y control del flujo de trabajo:

1. **Mantener la rama principal estable:** La rama `main` debe estar siempre en un estado ejecutable y libre de fallos. Desarrollar directamente en ella expone al proyecto a que subamos código incompleto o con errores que rompan la aplicación para el resto del equipo de forma inmediata.
2. **Validar los cambios con el pipeline de CI:** El flujo automatizado de integración continua (`ci.yml`) está configurado para ejecutarse con eventos de Pull Request hacia `main`. Al trabajar de forma aislada en la rama obligatoria `devops/ci-cd`, GitHub Actions puede levantar un entorno limpio, instalar dependencias con `npm ci`, correr las pruebas unitarias y verificar la compilación antes de mezclar los cambios.
3. **Control mediante Quality Gates:** Si subimos commits directamente a `main`, evadimos los filtros automáticos de calidad. Al usar un Pull Request, el pipeline actúa como un muro de contención: si una prueba falla o el proyecto no compila, el merge se bloquea, impidiendo que el código defectuoso contamine la rama estable.
4. **Trazabilidad e historial de cambios:** Integrar mediante ramas y Pull Requests obliga a dejar un registro claro de qué se modificó, qué pruebas se hicieron y por qué se subió el cambio, lo que mantiene el historial del repositorio ordenado y facilita la colaboración.

### Pregunta 2: ¿Qué problema se evita al utilizar `--skip-git` al crear el proyecto Angular?
Evita que el CLI de Angular cree un repositorio de Git independiente dentro de la subcarpeta `frontend/`. Si no usáramos este parámetro, tendríamos un repositorio anidado dentro de otro. Esto confunde al Git de la raíz, que suele interpretar la subcarpeta como un submódulo roto o directamente deja de rastrear su contenido, impidiéndonos subir el código del frontend al repositorio principal en GitHub de manera limpia.

### Pregunta 3: ¿Qué verifica `npm run build` en esta etapa del laboratorio?
Verifica que todo el código TypeScript de la aplicación de Angular compile a JavaScript estándar de forma exitosa, sin errores de sintaxis, tipos o importaciones rotas. Al mismo tiempo, simula el empaquetado final del software generando los archivos estáticos listos para producción en la carpeta `dist/`. Correrlo localmente nos asegura de que el build no va a romper el pipeline de CI una vez subido el código.

### Pregunta 4: ¿Qué utilidad tiene revisar `git status` o `git diff --cached` antes de realizar un commit?
`git status` nos permite ver un resumen rápido de qué archivos están modificados, cuáles están preparados para el commit y cuáles no están siendo rastreados. Esto es clave para no meter por error archivos basura o secretos (como claves de API o archivos `.env.local`). Por su parte, `git diff --cached` nos muestra la comparación línea por línea del código que ya preparamos con `git add`, permitiéndonos hacer una última revisión visual antes de confirmar los cambios de forma definitiva.

### Pregunta 5: ¿Qué evento activa el workflow `ci.yml`?
El workflow se activa de forma automática cuando se crea o se actualiza un **Pull Request** (`pull_request`) que tiene como destino la rama `main`. Esto garantiza que cualquier cambio propuesto en nuestra rama de trabajo sea evaluado antes de integrarse al código definitivo.

### Pregunta 6: En `runs-on: ubuntu-latest`, ¿qué representa `ubuntu-latest`?
Representa el entorno virtual o **runner** (una máquina virtual limpia alojada por GitHub) sobre la cual se va a ejecutar nuestro job. En este caso específico, le indica a GitHub Actions que monte y corra todos los pasos del flujo de validación sobre la versión estable más reciente de Ubuntu Linux.

### Pregunta 7: Ordene las etapas de validación que ejecuta el job frontend y explique por qué `npm ci` se ejecuta antes que las pruebas.

El orden secuencial de las etapas que ejecuta el job es:
1. **Obtener código** (`actions/checkout`)
2. **Configurar Node.js** (`actions/setup-node`) 
3. **Instalar dependencias** (`npm ci`) 
4. **Ejecutar pruebas** (`npm test`) 
5. **Construir Angular** (`npm run build`) 

**Justificación:**
`npm ci` se debe ejecutar antes de las pruebas porque el runner se inicializa como un entorno completamente limpio y vacío. Si intentáramos ejecutar las pruebas (`npm test`) antes de este paso, el comando fallaría de inmediato, ya que el sistema no tendría instalados los paquetes ni las dependencias necesarias para compilar y ejecutar el framework.

### Pregunta 8: Después del push, indique qué etapa del pipeline falla y qué ocurre con las etapas siguientes.

La etapa que falla es la de **"Ejecutar pruebas"** (`npm test`), ya que la aserción del test de Angular busca el texto incorrecto que configuramos a propósito. Como las acciones dentro de un job en GitHub Actions se ejecutan de manera secuencial, cuando un paso falla arrojando un error, el runner detiene el workflow de inmediato por seguridad. Esto significa que las etapas siguientes (en este caso, **"Construir Angular"**) se omiten y no llegan a ejecutarse.

### Pregunta 9: ¿Debería integrarse este Pull Request a main mientras el pipeline está fallando? Justifique.

No, no se debe integrar. El pipeline actúa como un **Quality Gate** (filtro de calidad automatizado) para asegurar que el software cumpla con los estándares mínimos antes de mezclarse. Si forzamos la integración de un Pull Request mientras el pipeline está fallando, subiríamos pruebas rotas o código defectuoso directamente a `main`, lo que rompería la rama principal para todo el equipo y violaría el acuerdo de mantenerla siempre ejecutable y libre de fallos.

### Pregunta 10: Clasificación de elementos

* **`package.json`**: versionable
* **`API_URL` pública**: variable/configuración
* **`AWS_REGION`**: variable/configuración
* **`DB_PASSWORD`**: secreto/no versionable
* **`API_TOKEN`**: secreto/no versionable
* **`terraform.tfstate`**: secreto/no versionable

### Pregunta 11: ¿Por qué una contraseña o token no debe escribirse directamente dentro de `ci.yml`, `cd.yml` o un archivo TypeScript del frontend?

1. Si escribimos claves directamente en los archivos del pipeline (`ci.yml` o `cd.yml`), estas quedan grabadas en texto plano dentro del historial del repositorio. Cualquier persona que tenga acceso a ver el repositorio podrá leerlas sin esfuerzo.
2. Si se pone un secreto en el código de Angular (TypeScript), este se compila y se envía directo al navegador del usuario final. Cualquier visitante podría abrir la consola del navegador (F12), revisar el código descargado y robar la credencial. Para evitar esto, las claves deben manejarse siempre en el backend o mediante variables de entorno y Secretos de GitHub.

### Pregunta 12: Si un secreto real fue incluido en un commit y luego se agrega su archivo a `.gitignore`, ¿queda solucionado el problema? Explique qué acción adicional debe realizarse.

No, no se soluciona. `.gitignore` solo evita que Git rastree el archivo en commits futuros. Pero como Git tiene memoria y registra todo el historial, el secreto seguirá existiendo en los commits anteriores del repositorio y cualquiera podría retroceder en el tiempo para extraerlo.

Para solucionarlo, se deben realizar obligatoriamente estas dos acciones:
1. **Rotar la credencial de inmediato:** Cambiar la contraseña o revocar el token expuesto para que la clave antigua quede inutilizable. 
2. **Purgar el historial del repositorio:** Utilizar herramientas especializadas de limpieza para reescribir el historial de Git y eliminar por completo el archivo del pasado del repositorio antes de volver a sincronizar con GitHub.

### Pregunta 13: ¿Qué diferencia existe entre `terraform validate`, `terraform plan` y `terraform apply`?

* Mientras que **`terraform validate`** realiza un chequeo puramente estático y local para asegurar que la sintaxis de nuestro código sea correcta sin conectarse a ningún lado, **`terraform plan`** simula el despliegue al comparar nuestro código con el estado actual en el proveedor, entregándonos una vista previa detallada de lo que se va a crear, modificar o destruir, pero sin alterar nada todavía.
* Por su parte, **`terraform apply`** se diferencia de los dos anteriores porque no es una prueba ni una simulación, es la acción definitiva que ejecuta esos cambios de manera real en la infraestructura y actualiza el archivo de estado (`.tfstate`).

### Pregunta 14: ¿Por qué `ci.yml` se activa con `pull_request` y `cd.yml` se activa con `push` sobre `main`?
*   **`ci.yml` (Integración Continua)** corre en el evento `pull_request` para funcionar como un filtro de calidad antes de integrar código nuevo. Nos permite validar que el build y los tests del frontend compilen sin errores en un entorno seguro antes de permitir mezclarlo con la rama estable.
*   **`cd.yml` (Despliegue Continuo)** se activa con el `push` directo en `main` (después de aprobar el merge del PR) porque el despliegue a un entorno operativo (como staging) solo debe ejecutarse de forma automática una vez que el código ha sido verificado, aprobado y consolidado definitivamente en la rama de producción.

### Pregunta 15: ¿Qué función cumple Terraform dentro de este flujo de CD?
En este flujo de CD específico, Terraform actúa como el orquestador que automatiza la preparación del entorno de staging simulado dentro del runner de GitHub Actions. Se encarga de limpiar de manera reproducible directorios antiguos, crear carpetas ordenadas y transferir limpiamente los archivos finales de distribución del frontend (`dist/`) generados en el build anterior. En un entorno de producción real, Terraform se encargaría de levantar y provisionar la infraestructura en la nube (como servidores, bases de datos o redes) de forma automatizada y controlada mediante código.

### Pregunta 16: ¿Por qué el workflow usa `${{ secrets.DEMO_TOKEN }}` en lugar de escribir el valor directamente?
Se usa para proteger la confidencialidad de la credencial y evitar fallos graves de seguridad. Si escribiéramos el token directamente en texto plano dentro del archivo `cd.yml`, este quedaría guardado para siempre de forma pública en el historial de commits de Git, permitiendo que cualquiera con acceso al repositorio pueda robarlo. Al usar `${{ secrets.DEMO_TOKEN }}`, GitHub cifra el secreto y solo lo inyecta de forma segura en la memoria temporal del runner durante el tiempo que dure la ejecución del pipeline, ocultándolo además con asteriscos en las pantallas de logs.

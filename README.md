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

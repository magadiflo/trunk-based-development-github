# Trunk Based Development y GitHub

- [Trunk Based Development (sitio de referencia)](https://trunkbaseddevelopment.com/)
    - Y su sección [You're doing it wrong](https://trunkbaseddevelopment.com/youre-doing-it-wrong)
- [How to Implement Trunk-Based Development: A Practical Guide (Unleash)](https://getunleash.io/trunk-based-development)
- [What Is Trunk-Based Development? A Complete Guide (Flagsmith)](https://flagsmith.com/blog/trunk-based-development)
- [Trunk-Based Development (Mergify)](https://mergify.com/learn/trunk-based-development)
- [Available rules for rulesets (GitHub Docs)](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets)
- [Git Credential Manager: varios usuarios (multiple-users.md)](https://github.com/git-ecosystem/git-credential-manager/blob/release/docs/multiple-users.md)
- [Managing your personal access tokens (GitHub Docs)](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)

> **Cómo leer esta documentación**
>
> - La práctica se hace en **GitHub real** con dos cuentas: **`magadiflo`** (dueño del repositorio, el desarrollador
>   principal) y **`dev-partner-lab`** (el compañero de equipo). El repositorio se llama
>   **`trunk-based-development-practice`**.
> - Los bloques `bash` muestran **quién** ejecuta cada comando con una etiqueta: `[magadiflo]` o
>   `[dev-partner-lab]`.
> - **Cada comando se explica al pie del bloque donde aparece por primera vez** (primero qué hace el comando completo y
>   luego cada una de sus partes). Si más adelante reaparece, ya no se vuelve a explicar; solo se explica lo nuevo
>   (por ejemplo, una opción que antes no se había usado). Los bloques de explicación empiezan con **📖 Comandos de
>   este bloque**.
> - Las salidas de terminal (`text`) provienen de **una simulación local** con los mismos usuarios, repositorio y
>   comandos. Los **hashes** (`93fd7e3`, etc.) y las fechas serán distintos en tu máquina. Como en la simulación el
>   remoto es una carpeta local y no GitHub, en tu terminal verás además líneas `remote:` que agrega GitHub (por
>   ejemplo, un enlace para crear el PR) y la URL `https://github.com/...` en vez de una ruta local.
> - Lo que es propio de GitHub (pantallas, rulesets, Actions, mensajes de rechazo del remoto) y de Git Credential
>   Manager está escrito a partir de la documentación y **no fue ejecutado**. Se indica con **(GitHub, no
>   verificado)**
>   o **(GCM, no verificado)**. Si algo no coincide con lo que ves, la interfaz habrá cambiado: ajústalo y anótalo.
> - Donde dice **📷 Captura sugerida**, conviene guardar una imagen en `./assets/` (como hiciste en GitFlow).

## ¿Qué es Trunk Based Development?

Trunk Based Development (TBD) es una estrategia de branching donde **todos los desarrolladores integran sus cambios en
una única rama compartida, el `trunk` (el "tronco"), que en Git es `main`**, con cambios **pequeños y frecuentes**, en
lugar de trabajar semanas en ramas largas.

La idea central no es "no usar ramas", sino **que las ramas vivan muy poco** (horas, a lo sumo uno o dos días) y que
`main` esté **siempre en un estado desplegable**.

> En GitFlow la pregunta es *"¿en qué rama va esto?"*. En TBD la pregunta es *"¿cómo lo hago lo bastante pequeño para
> integrarlo hoy sin romper nada?"*.

```text
GitFlow                                   Trunk Based Development

main     ──●────────────────●──           main  ──●──●──●──●──●──●──●──●──
develop  ──●──●──●──●──●──●──●──                    \/   \/    \/
feature     └──────●────┘                       (ramas de horas, se borran)
release              └──●──┘
```

### Principios

- Existe **una sola rama de vida infinita: `main`**. No hay `develop`.
- Los cambios se integran **al menos una vez al día**.
- `main` **siempre debe compilar y pasar las pruebas**. Se protege con integración continua (CI).
- El trabajo incompleto **no se esconde en una rama larga**: se integra a `main` **apagado detrás de un feature flag**.
- Los lanzamientos salen desde `main` (o desde una *release branch* corta).

### Estilos de TBD según el tamaño del equipo

| Estilo                                 | Cuándo se usa                               | Cómo se integra                                                                      |
|----------------------------------------|---------------------------------------------|--------------------------------------------------------------------------------------|
| **Commit directo al trunk**            | Equipos pequeños y de mucha confianza       | `git pull --rebase` → `git commit` → `git push` directo a `main`                     |
| **Ramas de vida corta + Pull Request** | Equipos medianos/grandes (el más común hoy) | Rama de menos de 1 día → PR pequeño → *Squash and merge* a `main` → se borra la rama |

En esta práctica se recorren **los dos estilos**: primero commits directos (Cambios `#1` y `#2`) y luego se **protege
`main`** y se trabaja con PR (Cambios `#3` al `#7`).

### Tipos de ramas

| Rama                   | Vida                   | Descripción                                                                                                                                                       |
|------------------------|------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `main` (trunk)         | Infinita               | Única fuente de verdad. Siempre desplegable.                                                                                                                      |
| `feature/*`, `fix/*`   | Horas, máximo 1-2 días | Una sola persona (o pareja). Se borra al integrarse.                                                                                                              |
| `release/*` (opcional) | Corta                  | Solo si se necesita una "foto" estable para producción. **No es una línea de desarrollo**: los arreglos se hacen primero en `main` y se llevan con `cherry-pick`. |

### Feature flags (feature toggles)

Si una funcionalidad tarda 2 semanas, **no** se deja 2 semanas en una rama. Se divide en pedazos pequeños que se
integran cada día, y el código incompleto queda **desactivado por un flag**:

```bash
if (nuevoCheckoutHabilitado) {
    return procesarConNuevoFlujo();   // en construcción
}
return procesarConFlujoActual();      // lo que ve producción
```

Así se **desacopla el despliegue (deploy) del lanzamiento (release)**: el código llega a producción, pero el usuario no
lo ve hasta que se enciende el flag.

### Requisitos para que funcione

Sin estas prácticas, TBD termina degradándose a "feature branching con otro nombre":

1. **CI rápida** en cada PR/commit (compilar + pruebas).
2. **Pruebas automatizadas** confiables.
3. **Feature flags** para código incompleto.
4. **Revisión de código ágil** (PR pequeños que se revisan en minutos, no en días).
5. **Cultura de commits pequeños.**

## Comparación: GitFlow vs Trunk Based Development

| Aspecto                      | GitFlow                                                           | Trunk Based Development                                    |
|------------------------------|-------------------------------------------------------------------|------------------------------------------------------------|
| Ramas de vida infinita       | `main` y `develop`                                                | Solo `main`                                                |
| Vida de las ramas de trabajo | Días o semanas (`feature`)                                        | Horas o 1-2 días                                           |
| Rama de integración          | `develop`                                                         | `main` directamente                                        |
| Preparar un lanzamiento      | Rama `release-*` con ajustes                                      | `main` ya está lista (o `release/*` corta con cherry-pick) |
| Hotfix                       | Rama `hotfix-*` → `main` **y** `develop`                          | Fix en `main` (+ cherry-pick a la release si aplica)       |
| Código incompleto            | Se queda en la rama `feature`                                     | Se integra apagado con feature flag                        |
| Riesgo de conflictos         | Alto (las ramas divergen mucho)                                   | Bajo (cambios pequeños y frecuentes)                       |
| Frecuencia de despliegue     | Por versiones planificadas                                        | Continua / varias veces al día                             |
| Exigencia de CI/CD y tests   | Moderada                                                          | **Alta (imprescindible)**                                  |
| Encaja mejor con             | Software con versiones calendarizadas (móvil, embebido, regulado) | Aplicaciones web/backend con despliegue continuo           |

### Cuándo conviene y cuándo no

- **Conviene:** servicios backend (Spring Boot, microservicios), equipos con CI/CD y buenas pruebas, despliegue
  frecuente.
- **No tanto:** software que se publica por calendario con revisiones largas (app stores) o que debe mantener varias
  versiones en paralelo durante años. Ahí las *release branches* de larga vida tienen sentido real.

## Origen y destino de las ramificaciones

| Rama                   | Origen                                                                                               | Destino                                                      | Convención          | Ejemplo                 |
|------------------------|------------------------------------------------------------------------------------------------------|--------------------------------------------------------------|---------------------|-------------------------|
| **main** (trunk)       |                                                                                                      |                                                              |                     |                         |
| **feature / fix**      | main                                                                                                 | main (PR + *Squash and merge*)                               | `feature/` · `fix/` | `feature/validar-email` |
| **release** (opcional) | main                                                                                                 | **no se fusiona de vuelta** (se borra al salir la siguiente) | `release/`          | `release/1.0`           |
| **hotfix**             | **No existe como rama aparte**: el arreglo nace en `main` y se lleva con `cherry-pick` a `release/*` |                                                              |                     |                         |

## Práctica: Tabla de cambios

| ID        | DESCRIPCIÓN CORTA                                               | RAMA DE TRABAJO                           | RAMA ORIGEN | RAMA DESTINO         | QUIÉN TRABAJA                               | QUIÉN REVISA    |
|-----------|-----------------------------------------------------------------|-------------------------------------------|-------------|----------------------|---------------------------------------------|-----------------|
| Cambio #1 | Endpoint `/api/saludo` (commit directo)                         | *(ninguna)*                               | main        | main                 | magadiflo                                   | —               |
| Cambio #2 | Conflicto: ambos editan `application.properties` (push directo) | *(ninguna)*                               | main        | main                 | magadiflo y dev-partner-lab                 | —               |
| Cambio #3 | Validar email al registrar usuario                              | `feature/validar-email`                   | main        | main                 | dev-partner-lab                             | magadiflo       |
| Cambio #4 | Nuevo checkout **incompleto** detrás de un feature flag         | `feature/nuevo-checkout`                  | main        | main                 | magadiflo                                   | dev-partner-lab |
| Cambio #5 | Dos PR en paralelo: el segundo entra en conflicto               | `feature/configurar-reintentos`           | main        | main                 | dev-partner-lab                             | magadiflo       |
| Cambio #6 | Preparar la versión 1.0                                         | `release/1.0`                             | main        | *(no vuelve a main)* | magadiflo                                   | —               |
| Cambio #7 | Bug en producción (v1.0.0) → v1.0.1                             | `fix/email-con-signo-mas` y `cherry-pick` | main        | main y `release/1.0` | dev-partner-lab (fix) y magadiflo (release) | magadiflo       |

> Entre los Cambios `#2` y `#3` se **activa la protección de `main`**. Por eso los dos primeros se hacen con commit
> directo y el resto con Pull Request.

## Inicio: preparación del entorno

### 1. Crear el repositorio en GitHub

Con la cuenta `magadiflo`:

> **GitHub > New repository**
> - **Repository name:** `trunk-based-development-practice`
> - **Visibility:** **Public** (los *rulesets* están disponibles en repositorios públicos con el plan gratuito)
> - **No** marcar *Add a README*, *.gitignore* ni *license* (el repositorio debe quedar vacío)

![01.png](assets/01.png)

### 2. Invitar a `dev-partner-lab` como colaborador

Con `magadiflo`:

> **Repositorio > Settings > Collaborators > Add people** → buscar `dev-partner-lab` → rol **Write**

Con `dev-partner-lab`: aceptar la invitación desde el correo, o entrando a:

```text
https://github.com/magadiflo/trunk-based-development-practice/invitations
```

![02.png](assets/02.png)

### 3. Dos cuentas en la misma PC: HTTPS y configuración local de Git

Si utilizas HTTPS, puedes trabajar con dos cuentas de GitHub desde la misma PC sin necesidad de generar y administrar
una llave SSH por cada cuenta.

La idea es mantener la configuración global de Git para tu cuenta principal (`magadiflo`) y establecer una configuración
local para el repositorio donde trabajarás como colaborador (`dev-partner-lab`). De esta manera, cada clon tendrá su
propia identidad de Git, sin afectar los demás repositorios de tu equipo.

#### Configuración global (`magadiflo`)

Si ya tienes configurada tu identidad principal, no necesitas modificar nada. Puedes comprobarla con:

```bash
C:\cmder
$ git config --global user.name
Martín Díaz

C:\cmder
$ git config --global user.email
tu-correo@ejemplo.com
```

> **📖 Comandos de este bloque**
>
> **`git config --global user.name`** y **`git config --global user.email`**: muestran la identidad que Git usa por
> defecto en **todos** tus repositorios.
> - `git config`: lee o escribe la configuración de Git. Si le das **solo la clave** (como aquí), **muestra** su valor;
>   si le das la clave **y un valor**, lo **guarda**.
> - `--global`: indica que se trabaja con la configuración **de tu usuario del sistema** (archivo `.gitconfig` de tu
>   carpeta personal), que vale para todos los repositorios. Los otros niveles son `--system` (toda la PC) y `--local`
>   (un solo repositorio).
> - `user.name` / `user.email`: las claves con el nombre y el correo que Git escribe como **autor** de cada commit.

#### Configuración local (`dev-partner-lab`)

Cuando clones el repositorio para trabajar como colaborador, configurarás su identidad únicamente dentro de ese
repositorio:

```bash
$ git config --local user.name "dev-partner-lab"
$ git config --local user.email "martin.g.diaz.flores@gmail.com"
```

> **📖 Comandos de este bloque**
>
> **`git config --local user.name "dev-partner-lab"`** y
> **`git config --local user.email "martin.g.diaz.flores@gmail.com"`**: guardan la identidad de `dev-partner-lab` **solo
> en este repositorio**.
> - `--local`: el valor se guarda en el archivo `.git/config` del repositorio actual. Tiene **prioridad** sobre
>   `--global`, así que dentro de este clon manda la identidad local y en los demás proyectos sigue la global. (Orden
>   de prioridad: local → global → system.)
> - `"dev-partner-lab"` / `"martin.g.diaz.flores@gmail.com"`: ahora sí hay **valor**, por eso el comando **escribe** en
>   lugar de mostrar. Si el correo no pertenece a la cuenta, GitHub mostrará los commits sin el usuario asociado.

> `git config --local` no modifica la configuración global ni afecta a los demás proyectos.

#### ¿Cómo funciona la autenticación con HTTPS?

La identidad configurada mediante `user.name` y `user.email` determina quién aparece como **autor de los commits**, pero
**no determina qué cuenta de GitHub realiza el push**. De hecho, la documentación de Git Credential Manager lo advierte:
`user.name` y `user.email` no afectan a GCM.

Para autenticar las operaciones HTTPS utilizaremos **Git Credential Manager (GCM)**, que gestiona las credenciales de
GitHub:

- `magadiflo` seguirá utilizando su autenticación habitual.
- `dev-partner-lab` deberá autenticarse con sus propias credenciales de GitHub.

**El problema:** si no se le indica nada, Git le pide credenciales a GCM **sin decirle qué usuario quiere** (para Git,
el usuario del remoto queda vacío). Si GCM ya tiene guardada tu cuenta principal, **puede reutilizarla**, y entonces el
push de `dev-partner-lab` saldría como `magadiflo` sin darte ningún error: los commits dirían `dev-partner-lab`, pero
quien empuja sería `magadiflo`. Con dos cuentas en el mismo dominio (`github.com`) hay que indicar el usuario de forma
explícita.

#### Indicar a GCM qué cuenta usa cada clon

GCM documenta dos formas de hacerlo: poner el usuario en la URL del remoto, o configurar
`credential.https://github.com.username`. Aquí usamos la segunda, **de forma local en cada clon**, porque encaja con la
idea de "configurar el repositorio local":

```bash
# en el clon de magadiflo
$ git config --local credential.https://github.com.username "magadiflo"

# en el clon de dev-partner-lab
$ git config --local credential.https://github.com.username "dev-partner-lab"
```

> **📖 Comandos de este bloque**
>
> `git config --local credential.https://github.com.username "..."`: le dice a Git qué **usuario de GitHub** debe
> pedirle a GCM al hablar con `https://github.com` desde este repositorio.
> - `credential.https://github.com.username`: es una clave con el formato `credential.<URL>.username`. El `<URL>` (aquí
>   `https://github.com`) limita el efecto a ese sitio: para otros servidores (GitLab, etc.) no se envía ningún
>   usuario.
> - `"dev-partner-lab"` / `"magadiflo"`: el usuario que GCM debe buscar o crear. Cada usuario tiene sus propias
>   credenciales guardadas, así que **ya no se mezclan**.

> Se configura la cuenta de **magadiflo** también, aunque sea la principal: cuando GCM tenga **dos** cuentas para
> `github.com`, un clon sin usuario definido es ambiguo.

> **Alternativa equivalente:** poner el usuario en la URL,
> `https://dev-partner-lab@github.com/magadiflo/trunk-based-development-practice.git`. Funciona igual, pero deja el
> usuario visible dentro de la URL del remoto.

#### Primer push de `dev-partner-lab`: iniciar sesión con la cuenta correcta

La primera vez que `dev-partner-lab` empuje, GCM no encontrará credenciales para ese usuario y abrirá su ventana de
inicio de sesión de GitHub.

Precauciones:

- Si tu navegador predeterminado tiene abierta la sesión de **magadiflo**, autoriza con **dev-partner-lab**: abre el
  enlace en una ventana de incógnito o elige la opción de **código de dispositivo** (*device code*), que puedes
  completar en un navegador donde esté la otra cuenta.
- Si prefieres un token, debe ser un **token clásico (*classic*) con el permiso `repo`**. Según GitHub Docs, los tokens
  de grano fino (*fine-grained*) todavía no cubren el caso de contribuir a repositorios donde eres colaborador de otra
  cuenta personal.

Para comprobar qué cuentas guardó GCM:

```bash
$ git credential-manager github list
```

> **📖 Comandos de este bloque**
>
> `git credential-manager github list`: lista las cuentas de GitHub que Git Credential Manager tiene guardadas.
> - `git credential-manager`: invoca a GCM desde Git.
> - `github`: el subcomando para el proveedor GitHub (también existen `login` y `logout` para iniciar o cerrar sesión de
>   una cuenta; mira `git credential-manager github --help` para sus opciones).
> - `list`: muestra las cuentas guardadas. Tras el primer push de `dev-partner-lab` deberían aparecer **las dos**.

No es necesario configurar una llave SSH por cuenta ni modificar la configuración global de Git para establecer las
identidades locales.

### 4. Clonar dos copias del repositorio

Cada "desarrollador" tendrá su propia carpeta y su propio clon del repositorio. De esta manera, podremos simular el
trabajo de dos integrantes de un equipo desde una misma PC.

```text
M:\practice\
├── magadiflo\
│   └── trunk-based-development-practice\      ← clon de magadiflo
└── dev-partner-lab\
    └── trunk-based-development-practice\      ← clon de dev-partner-lab
```

#### `[magadiflo]`

Ubicados en el directorio `M:\practice` mediante el CMD (usaré para este usuario el `Cmder`), realizamos los siguientes
pasos para preparar nuestro repositorio local y clonar el repositorio utilizando HTTPS:

```bash
M:\practice
$ mkdir magadiflo && cd magadiflo

M:\practice\magadiflo
$ git clone https://github.com/magadiflo/trunk-based-development-practice.git
Cloning into 'trunk-based-development-practice'...
warning: You appear to have cloned an empty repository.

M:\practice\magadiflo
$ cd trunk-based-development-practice\

M:\practice\magadiflo\trunk-based-development-practice (main -> origin)
$ git config --local credential.https://github.com.username "magadiflo"
```

> **📖 Comandos de este bloque**
>
> `mkdir magadiflo && cd magadiflo`: crea una carpeta y entra en ella.
> - `mkdir magadiflo`: *make directory*, crea la carpeta `magadiflo`.
> - `&&`: ejecuta el comando siguiente **solo si el anterior terminó bien**.
> - `cd magadiflo`: *change directory*, cambia la carpeta actual a `magadiflo`.
>
> `git clone https://github.com/magadiflo/trunk-based-development-practice.git`: descarga una copia completa del
> repositorio remoto.
> - `git clone`: crea una carpeta con el nombre del repositorio, descarga todo su historial, define el remoto `origin`
>   apuntando a esa URL y deja listo el seguimiento de `main`.
> - `https://github.com/...git`: la URL HTTPS del repositorio (se copia del botón **Code** de GitHub).
>
> **`cd trunk-based-development-practice`**: entra en la carpeta recién clonada (todos los comandos Git siguientes se
> ejecutan ahí dentro).

Si la identidad global ya está configurada para `magadiflo`, no necesitamos configurar `user.name` ni `user.email`. Lo
comprobamos mostrando **de dónde viene** cada valor:

```bash
M:\practice\magadiflo\trunk-based-development-practice (main -> origin)
$ git config --show-origin user.name
file:C:/Users/magadiflo/.gitconfig      Martín Díaz

M:\practice\magadiflo\trunk-based-development-practice (main -> origin)
$ git config --show-origin user.email
file:C:/Users/magadiflo/.gitconfig      magadiflo@gmail.com
```

> **📖 Comandos de este bloque**
>
> `git config --show-origin user.name`: muestra el valor que Git usará **y el archivo donde está definido**.
> - `--show-origin`: antepone la ruta del archivo de configuración. Así distingues si el valor viene de `.git/config`
>   (local) o del `.gitconfig` global. (Sin esta opción solo verías el valor.)
> - `user.name` / `user.email`: las claves que se consultan.

#### `[dev-partner-lab]`

Clonamos otra copia del mismo repositorio (el repositorio sigue perteneciendo a `magadiflo`). Así que nos posicionamos
en el directorio `M:\practice` y esta vez usaremos una línea de comandos distinta, para este usuario usaremos
`GitBash`.

````bash
magadiflo@SysEngJava UCRT64 /m/practice
$ mkdir dev-partner-lab && cd dev-partner-lab

magadiflo@SysEngJava UCRT64 /m/practice/dev-partner-lab
$ git clone https://github.com/magadiflo/trunk-based-development-practice.git
Cloning into 'trunk-based-development-practice'...
warning: You appear to have cloned an empty repository.

magadiflo@SysEngJava UCRT64 /m/practice/dev-partner-lab
$ cd trunk-based-development-practice/ 
````

Ahora configuramos la identidad local del colaborador y la cuenta que usará GCM:

```bash
magadiflo@SysEngJava UCRT64 /m/practice/dev-partner-lab/trunk-based-development-practice (main)
$ git config --local user.name "dev-partner-lab"

magadiflo@SysEngJava UCRT64 /m/practice/dev-partner-lab/trunk-based-development-practice (main)
$ git config --local user.email "martin.g.diaz.flores@gmail.com"

magadiflo@SysEngJava UCRT64 /m/practice/dev-partner-lab/trunk-based-development-practice (main)
$ git config --local credential.https://github.com.username "dev-partner-lab"
```

Verificamos que se hayan aplicado correctamente los datos, y **de dónde** vienen:

```bash
magadiflo@SysEngJava UCRT64 /m/practice/dev-partner-lab/trunk-based-development-practice (main)
$ git config --show-origin user.name
file:.git/config        dev-partner-lab

magadiflo@SysEngJava UCRT64 /m/practice/dev-partner-lab/trunk-based-development-practice (main)
$ git config --show-origin user.email
file:.git/config        martin.g.diaz.flores@gmail.com

magadiflo@SysEngJava UCRT64 /m/practice/dev-partner-lab/trunk-based-development-practice (main)
$ git config --local --get-regexp "^(user|credential)\."
user.name dev-partner-lab
user.email martin.g.diaz.flores@gmail.com
credential.https://github.com.username dev-partner-lab

magadiflo@SysEngJava UCRT64 /m/practice/dev-partner-lab/trunk-based-development-practice (main)
$ git remote -v
origin  https://github.com/magadiflo/trunk-based-development-practice.git (fetch)
origin  https://github.com/magadiflo/trunk-based-development-practice.git (push)
```

> **📖 Comandos de este bloque**
>
> `git config --local --get-regexp "^(user|credential)\."`: lista las claves de la configuración **local** cuyo
> nombre empieza por `user.` o `credential.`.
> - `--get-regexp`: busca claves cuyo nombre coincide con una **expresión regular** y muestra `clave valor`.
> - `"^(user|credential)\."`: la expresión regular: `^` = al inicio del nombre; `(user|credential)` = una de las dos
>   palabras; `\.` = un punto literal.
>
> `git remote -v`: muestra los repositorios remotos configurados y sus URL.
> - `git remote`: administra los remotos. Sin más argumentos, solo lista sus nombres (`origin` es el que crea
>   `git clone`).
> - `-v`: *verbose*, agrega la URL de cada remoto, una para bajar cambios (*fetch*) y otra para subirlos (*push*).

> **Importante:** el repositorio sigue perteneciendo a `magadiflo`. Sin embargo, `dev-partner-lab` podrá contribuir
> porque previamente habrá aceptado la invitación como colaborador.

> **Sobre la autenticación:** cuando `dev-partner-lab` realice su primer push, GCM deberá utilizar las credenciales de
> esa cuenta
> (ver [Primer push de dev-partner-lab](#primer-push-de-dev-partner-lab-iniciar-sesión-con-la-cuenta-correcta)).

### 5. Proyecto base Spring Boot + CI (commit inicial)

El proyecto base se genera una sola vez y lo sube `magadiflo`.

**Paso 1: generar el proyecto** en
[start.spring.io](https://start.spring.io/#!type=maven-project&language=java&platformVersion=4.1.2&packaging=jar&configurationFileFormat=yaml&jvmVersion=25&groupId=dev.magadiflo&artifactId=trunk-based-development-practice&packageName=dev.magadiflo.tbd&dependencies=web,lombok):

| Campo        | Valor                                             |
|--------------|---------------------------------------------------|
| Project      | Maven                                             |
| Language     | Java                                              |
| Spring Boot  | 4.1.2                                             |
| Group        | `dev.magadiflo`                                   |
| Artifact     | `trunk-based-development-practice`                |
| Package name | `dev.magadiflo.tbd`                               |
| Packaging    | Jar                                               |
| Java         | 25 *(o la que uses; debe coincidir con `ci.yml`)* |
| Dependencies | Spring Web, Lombok                                |

Descomprime el `.zip` y copia **su contenido** (`pom.xml`, `mvnw`, `mvnw.cmd`, `.mvn/`, `.gitignore`, `src/`) dentro del
clon de `magadiflo`.

**Paso 2: el workflow de CI.** El archivo se crea dentro del proyecto, en la raíz del clon de `magadiflo` (la carpeta
`trunk-based-development-practice`, junto a `pom.xml`). GitHub solo reconoce los workflows que estén exactamente en
`.github/workflows/`. Si esas carpetas no existen, créalas:

````bash
M:\practice\magadiflo\trunk-based-development-practice (main -> origin)
$ mkdir .github\workflows 
````

> **📖 Comandos de este bloque**
>
> `mkdir .github\workflows`: crea las dos carpetas anidadas de una sola vez.
> - `mkdir`: ya explicado.

Dentro de esa carpeta crearemos el archivo `ci.yml`. El nombre es solo una convención: puede llamarse como quieras
(`build.yml`, `pruebas.yml`, etc.), siempre que termine en `.yml` o `.yaml`. Aun así, conviene que sea semántico, es
decir, que el nombre diga qué hace el workflow (`ci.yml` para integración continua, `deploy.yml` para despliegue),
porque en un proyecto real habrá varios archivos en esa carpeta y así se distinguen de un vistazo.

Lo que sí debes respetar es lo que hay dentro del archivo: el identificador del job (`build`) es el nombre del check
que más adelante exigiremos en el `ruleset`. Si lo cambias, también tendrías que actualizar el ruleset.

````bash
M:\practice\magadiflo\trunk-based-development-practice (main -> origin)
$ type nul > .github\workflows\ci.yml

M:\practice\magadiflo\trunk-based-development-practice (main -> origin)
$ cd .github\workflows

M:\practice\magadiflo\trunk-based-development-practice\.github\workflows (main -> origin)
$ ls -l
total 0
-rw-r--r-- 1 magadiflo 197121 0 Nov 16 23:07 ci.yml
````

El archivo `ci.yml` tendrá el siguiente contenido:

```yaml
name: CI

on:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]
  merge_group:            # solo necesario si algún día usas merge queue

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - uses: actions/setup-java@v6
        with:
          distribution: temurin
          java-version: '25'
          cache: maven
      - name: Compilar y ejecutar pruebas
        run: ./mvnw -B verify
```

> **📖 Comandos de este bloque**
>
> `./mvnw -B verify`: compila el proyecto, ejecuta las pruebas y valida el resultado, sin pedir nada por consola.
> - `./mvnw`: el *Maven Wrapper* del proyecto; usa la versión de Maven definida en el proyecto sin tener que instalarla
>   (`./` = "el archivo de esta carpeta").
> - `-B`: *batch mode*, modo no interactivo con salida más limpia, pensado para servidores de CI.
> - `verify`: fase del ciclo de vida de Maven. Ejecuta todas las anteriores (compilar, probar, empaquetar) y luego las
>   verificaciones finales.

> El job se llama `build`: ese será el nombre del *check* que luego exigiremos en el ruleset.

**Versiones usadas y cómo verificarlas.** Consultadas el 16 de noviembre de 2026 en las páginas de *Releases*:
`actions/checkout` **v7** (última publicada: v7.0.1) y `actions/setup-java` **v6** (última publicada: v6.1.0). Las
versiones cambian con el tiempo; para comprobarlas:

1. Abre [`actions/checkout/releases`](https://github.com/actions/checkout/releases) y [
   `actions/setup-java/releases`](https://github.com/actions/setup-java/releases) y busca la etiqueta **Latest**. En la
   web ves la versión completa (`v7.0.1`); en el workflow se escribe solo la **mayor** (`@v7`).
2. Lee las notas de la versión por si hay cambios incompatibles marcados como **[BREAKING]**. En `checkout`, el cambio
   de comportamiento de v7 afecta a los eventos `pull_request_target` y `workflow_run`, que este workflow **no usa**
   (usa `pull_request` y `push`).
3. Revisa el `README.md` de cada action: sus ejemplos muestran la versión mayor vigente.
4. Para no hacerlo a mano, **Dependabot** puede avisarte con un PR.

**Sobre Java:** `java-version: '25'` debe coincidir con la versión de tu proyecto (`<java.version>` en el `pom.xml`) y
con la que usas en tu máquina (`java -version`). `temurin` es la distribución de Java (Eclipse Temurin); el README de
`setup-java` v6 acepta versiones mayores como `'25'`. `cache: maven` guarda las dependencias de Maven entre ejecuciones
para que la CI sea más rápida.

**Paso 3: el permiso de ejecución de `mvnw`.** Al descomprimir en Windows, `mvnw` suele subirse **sin permiso de
ejecución** y en el runner Linux de GitHub la CI fallaría con `Permission denied`. Se corrige en el índice de Git:

`[magadiflo]`

```bash
M:\practice\magadiflo\trunk-based-development-practice (main -> origin)
$ git add .
warning: in the working copy of '.gitattributes', LF will be replaced by CRLF the next time Git touches it
warning: in the working copy of '.gitignore', LF will be replaced by CRLF the next time Git touches it
warning: in the working copy of '.mvn/wrapper/maven-wrapper.properties', LF will be replaced by CRLF the next time Git touches it
warning: in the working copy of 'mvnw.cmd', LF will be replaced by CRLF the next time Git touches it
warning: in the working copy of 'pom.xml', LF will be replaced by CRLF the next time Git touches it
warning: in the working copy of 'src/main/java/dev/magadiflo/tbd/TrunkBasedDevelopmentPracticeApplication.java', LF will be replaced by CRLF the next time Git touches it
warning: in the working copy of 'src/main/resources/application.yaml', LF will be replaced by CRLF the next time Git touches it
warning: in the working copy of 'src/test/java/dev/magadiflo/tbd/TrunkBasedDevelopmentPracticeApplicationTests.java', LF will be replaced by CRLF the next time Git touches it

M:\practice\magadiflo\trunk-based-development-practice (main -> origin)
$ git update-index --chmod=+x mvnw

M:\practice\magadiflo\trunk-based-development-practice (main -> origin)
$ git ls-files -s mvnw
100755 bd8896bf2217b46faa0291585e01ac1a3441a958 0       mvnw
```

> **📖 Comandos de este bloque**
>
> `git add .`: prepara (*stage*) todos los cambios de la carpeta actual para el próximo commit.
> - `git add`: mueve cambios al *staging area*, la "lista" de lo que entrará en el commit.
> - `.`: significa "la carpeta actual y todo lo que contiene". Respeta el `.gitignore`.
>
> `git update-index --chmod=+x mvnw`: marca `mvnw` como ejecutable **dentro de Git**.
> - `git update-index`: comando de bajo nivel que modifica el índice (el *staging area*) directamente.
> - `--chmod=+x`: agrega el permiso de ejecución. Se hace en el índice porque en Windows el sistema de archivos no lo
>   guarda como en Linux.
> - `mvnw`: el archivo al que se aplica.
>
> `git ls-files -s mvnw`: muestra cómo está registrado `mvnw` en el índice.
> - `git ls-files`: lista los archivos que Git está siguiendo.
> - `-s` (*--stage*): agrega modo, hash y número de etapa de cada archivo.
> - `mvnw`: limita la salida a ese archivo.
> - El `100755` de la salida significa "archivo ejecutable" (antes era `100644`, archivo normal).



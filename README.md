# Git desde cero: charla-taller

Apuntes de la charla-taller de Git del **13 de noviembre de 2026**, organizada en 0xDecode.
Aquí tienes todo lo que hacemos en directo, en orden, para seguirlo o repetirlo en casa.

> **¿Cómo se usa este repositorio?**
> Esto es la **teoría y la guía paso a paso**. La práctica en equipo está en otro repo:
> [`ejercicios-equipo`](https://github.com/skillIssu3/ejercicios-equipo).

No hace falta saber Git, ni terminal. Si sabes algo de Python y usas VS Code, vas sobrado.

---

## Índice

0. [Antes de empezar](#0-antes-de-empezar)
1. [Qué es Git y qué es GitHub](#1-qué-es-git-y-qué-es-github)
2. [La terminal en 6 comandos](#2-la-terminal-en-6-comandos)
3. [Instalar y configurar Git](#3-instalar-y-configurar-git)
4. [SSH: cómo funciona y cómo configurarlo](#4-ssh-cómo-funciona-y-cómo-configurarlo)
5. [Tu primer repositorio](#5-tu-primer-repositorio)
6. [Trabajar con otras personas](#6-trabajar-con-otras-personas)
7. [Ramas y Pull Requests](#7-ramas-y-pull-requests)
8. [Chuleta de comandos](#8-chuleta-de-comandos)
9. [Errores habituales](#9-errores-habituales)

---

## 0. Antes de empezar

Todo el taller se hace con **Git en la terminal de Linux**.

- **Si usas Linux:** ya lo tienes, salta al punto 1.
- **Si usas Windows:** instala **WSL** (Linux dentro de Windows). Abre PowerShell **como administrador** y ejecuta:
  ```
  wsl --install
  ```
  Reinicia el equipo. Al volver a abrir "Ubuntu" te pedirá crear un usuario y una contraseña.
- **VS Code (recomendado):** instálalo y añade la extensión **WSL**. Así podrás abrir carpetas de Linux con `code .`.

> **Importante en WSL:** trabaja siempre dentro de tu carpeta personal (`~/`), **nunca** en `/mnt/c`.
> Y la clave SSH del punto 4 se genera **dentro de WSL**, no en Windows.

También necesitas una **cuenta de GitHub** ([github.com/signup](https://github.com/signup)).

---

## 1. Qué es Git y qué es GitHub

| | Git | GitHub |
|---|---|---|
| **Qué es** | Programa de terminal | Web donde se alojan repositorios |
| **Dónde vive** | En tu ordenador | En internet |
| **Para qué** | Guardar el historial de versiones de tu código | Compartirlo y trabajar en equipo |
| **Internet** | No lo necesita | Sí |

**Git** es como un sistema de puntos de guardado para tu código: cada *commit* es una foto del proyecto en un momento dado, a la que puedes volver.
**GitHub** es un servidor donde subes esas fotos para guardarlas y compartirlas.

Git nació en 2005, cuando Linus Torvalds (el creador de Linux) perdió la licencia gratuita de la herramienta que usaba y escribió la suya en pocos días.

---

## 2. La terminal en 6 comandos

| Comando | Qué hace |
|---|---|
| `pwd` | Muestra en qué carpeta estás |
| `ls` | Lista lo que hay en la carpeta |
| `cd carpeta` | Entra en una carpeta (`cd ..` sube una) |
| `mkdir nombre` | Crea una carpeta |
| `cat archivo` | Muestra el contenido de un archivo |
| `clear` | Limpia la pantalla |

Truco: pulsa **Tab** para autocompletar nombres, y **↑** para repetir el comando anterior.

---

## 3. Instalar y configurar Git

**Instalar** (en Ubuntu/Debian/WSL):

```
sudo apt update && sudo apt install -y git
```

> Otras distribuciones: Fedora `sudo dnf install -y git`, Arch `sudo pacman -S git`.

Comprueba que funciona:

```
git --version
```

**Configurar** tu identidad (el correo debe ser el de tu cuenta de GitHub):

```
git config --global user.name "Tu Nombre"
git config --global user.email "tu-email@ejemplo.com"
git config --global init.defaultBranch main
git config --global pull.rebase false
git config --global core.editor "nano"
```

Qué hace cada línea:

- `user.name` y `user.email`: quién firma tus commits.
- `init.defaultBranch main`: la rama principal se llamará `main`.
- `pull.rebase false`: define cómo Git junta cambios al hacer `git pull`. **Sin esto, Git da error** cuando tú y un compañero habéis subido cosas distintas.
- `core.editor "nano"`: el editor que Git abre cuando necesita que escribas un mensaje.

Para ver lo que has configurado: `git config --global --list`

---

## 4. SSH: cómo funciona y cómo configurarlo

### La idea

Para subir código a GitHub tienes que demostrar que eres tú. En vez de una contraseña, usamos un **par de claves**:

- **Clave privada** (`id_ed25519`): se queda en tu ordenador. **Nunca se comparte ni se sube.**
- **Clave pública** (`id_ed25519.pub`): se la das a GitHub. No pasa nada si alguien la ve.

### Qué ocurre por detrás

1. Tú le dices a GitHub: "quiero entrar".
2. GitHub, que tiene guardada tu clave pública, te manda un **reto** (un dato aleatorio).
3. Tu ordenador **firma el reto con tu clave privada**.
4. GitHub comprueba la firma con tu clave pública. Si encaja, sabe que eres tú.

La clave privada **nunca viaja por internet** y no se envía ninguna contraseña. Por eso es más seguro, y no tienes que escribir nada cada vez que haces `git push`.

### Configurarlo

**1. Genera el par de claves:**

```
ssh-keygen -t ed25519 -C "tu-email@ejemplo.com"
```

Pulsa **Enter** a todo (ruta por defecto). La contraseña (*passphrase*) es opcional.

**2. Muestra tu clave pública** y copia todo lo que aparezca:

```
cat ~/.ssh/id_ed25519.pub
```

> **ATENCIÓN:** copia solo la que termina en **`.pub`**. La otra (`id_ed25519`, sin `.pub`) es la privada y **no se comparte nunca**.

**3. Súbela a GitHub:** *Settings > SSH and GPG keys > New SSH key*. Ponle un título (por ejemplo, "Portátil WSL") y pega la clave.

**4. Comprueba que funciona:**

```
ssh -T git@github.com
```

La primera vez, SSH te preguntará esto:

```
The authenticity of host 'github.com' can't be established.
Are you sure you want to continue connecting (yes/no)?
```

Es normal: es tu ordenador comprobando que GitHub es GitHub. Puedes comparar la huella que muestra con la oficial en la
[documentación de GitHub](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints), y escribir `yes`.

Si todo va bien, verás:

```
Hi TU_USUARIO! You've successfully authenticated, but GitHub does not provide shell access.
```

> **Recuerda:** al clonar o vincular repositorios usa siempre la URL **SSH** (`git@github.com:...`), no la de HTTPS.

---

## 5. Tu primer repositorio

**Paso 0:** en la web de GitHub, crea un repositorio **vacío** (sin README ni `.gitignore`) y copia su URL SSH.

**1. Crea la carpeta e inicializa Git:**

```
mkdir -p ~/git
cd ~/git
mkdir mi-proyecto
cd mi-proyecto
git init
```

**2. Crea un archivo y haz tu primer commit:**

```
echo "# Mi proyecto" > README.md
git status
git add .
git commit -m "Commit inicial"
```

El flujo de siempre es este:

```
editar archivos  →  git add  →  git commit  →  git push
                    (preparar)   (guardar)      (subir)
```

**3. Conecta con GitHub y sube:**

```
git remote add origin git@github.com:TU_USUARIO/mi-proyecto.git
git push -u origin main
```

Refresca la página del repositorio en GitHub: ahí está tu código.

---

## 6. Trabajar con otras personas

**Clonar** un repositorio (descargarlo con todo su historial):

```
git clone git@github.com:USUARIO/PROYECTO.git
cd PROYECTO
```

**Traer los cambios de los demás:**

```
git pull
```

**El problema típico:** tú y un compañero habéis hecho commits distintos. El primero en hacer `git push` lo consigue. El segundo ve:

```
! [rejected]  main -> main (fetch first)
```

No es grave: el servidor tiene cosas que tú no tienes. Haz `git pull` para traerlas y vuelve a hacer `git push`.

Si se abre un editor pidiendo un mensaje de *merge*, guarda y cierra (en nano: `Ctrl+O`, `Enter`, `Ctrl+X`).

### Conflictos

Si dos personas cambian **la misma línea** del mismo archivo, Git no sabe cuál elegir y te lo marca así:

```
<<<<<<< HEAD
const puerto = 8080;
=======
const puerto = 3000;
>>>>>>> main
```

- Lo de arriba (`HEAD`) es **tu versión**. Lo de abajo es **la de tu compañero**.
- Edita el archivo, **deja una sola versión** (o mezcla las dos) y **borra las tres líneas de marcas** (`<<<<<<<`, `=======`, `>>>>>>>`).
- En VS Code aparecen botones: *Accept Current Change*, *Accept Incoming Change* y *Accept Both Changes*.

Después, termina el merge:

```
git add archivo
git commit -m "Resuelvo el conflicto"
git push
```

> Si te lías, `git merge --abort` cancela el merge y vuelves a como estabas.

---

## 7. Ramas y Pull Requests

Una **rama** es una línea de trabajo independiente. Sirve para probar algo sin romper `main`, que es lo que ven y usan todos.

**Crear una rama y cambiar a ella:**

```
git checkout -b mi-rama
```

**Trabajar y subir tu rama:**

```
git add .
git commit -m "Añado la función X"
git push -u origin mi-rama
```

**Pull Request (desde la web):**

1. Entra en el repositorio en GitHub. Verás un botón verde **Compare & pull request**.
2. Escribe un título y una descripción de lo que has hecho.
3. Un compañero revisa los cambios en *Files changed* y, si está bien, pulsa **Merge pull request**.

**Volver a `main` y traer los cambios:**

```
git checkout main
git pull
```

---

## 8. Chuleta de comandos

| Comando | Para qué sirve |
|---|---|
| `git status` | Estado del proyecto: qué ha cambiado y qué está preparado |
| `git add .` | Prepara todos los cambios para el commit |
| `git commit -m "mensaje"` | Guarda los cambios preparados |
| `git push` | Sube tus commits a GitHub |
| `git pull` | Trae y junta los cambios de GitHub |
| `git clone URL` | Descarga un repositorio |
| `git log --oneline --graph` | Historial resumido en forma de árbol |
| `git diff` | Qué has cambiado y no has preparado todavía |
| `git restore archivo` | Descarta los cambios no guardados de un archivo |
| `git remote -v` | A qué repositorio remoto estás conectado |
| `git branch` | Lista de ramas (la actual lleva `*`) |
| `git checkout -b nombre` | Crea una rama y cambia a ella |
| `git checkout nombre` | Cambia a una rama existente |

---

## 9. Errores habituales

- **`Permission denied (publickey)`**
  Tu clave SSH no está subida a GitHub, o estás usando una URL `https://` en vez de `git@github.com:`.
  Comprueba con `ssh -T git@github.com` y con `git remote -v`.

- **`Repository not found` al clonar**
  La URL tiene un error, o el repositorio es privado y no tienes acceso (por ejemplo, aún no has aceptado la invitación).

- **`Need to specify how to reconcile divergent branches`**
  Falta `git config --global pull.rebase false`.

- **`! [rejected] ... (fetch first)`**
  Hay cambios en GitHub que no tienes. `git pull` y luego `git push`.

- **`fatal: not a git repository`**
  No estás dentro de una carpeta con Git. Haz `pwd` y `ls`, y entra con `cd` en la carpeta del proyecto.

- **`fatal: remote origin already exists`**
  Ya tenías un remoto configurado. Cámbialo con `git remote set-url origin URL`.

---

## Siguiente paso

Ahora, a practicar: clona [`ejercicios-equipo`](https://github.com/skillIssu3/ejercicios-equipo) y sigue las rondas con tu grupo.

# Activar verificación de commits en GitHub

Esta guía explica cómo configurar Git en Windows para firmar los commits mediante una clave SSH y conseguir que GitHub los muestre como **Verified**.

## 1. Verificar si existe una clave SSH

Desde PowerShell:

```powershell
Get-ChildItem ~/.ssh
```

Si aparece un error indicando que la carpeta `.ssh` no existe, significa que todavía no se ha creado una clave SSH.

---

## 2. Crear una clave SSH

Ejecutar:

```powershell
ssh-keygen -t ed25519 -C "correo@ejemplo.com"
```

Git preguntará dónde guardar la clave:

```text
Enter file in which to save the key:
```

Presionar **Enter** para utilizar la ubicación predeterminada:

```text
C:\Users\USUARIO\.ssh\id_ed25519
```

Después solicitará una contraseña:

```text
Enter passphrase (empty for no passphrase):
```

Se recomienda utilizar una contraseña para proteger la clave privada.

Se crearán dos archivos:

```text
.ssh/
├── id_ed25519       ← Clave privada
└── id_ed25519.pub   ← Clave pública
```

> **IMPORTANTE:** Nunca compartir el archivo `id_ed25519`.  
> El archivo que se registra en GitHub es `id_ed25519.pub`.

---

## 3. Obtener la clave pública

Ejecutar:

```powershell
Get-Content ~/.ssh/id_ed25519.pub
```

Se mostrará algo parecido a:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... correo@ejemplo.com
```

Copiar la línea completa.

---

## 4. Registrar la clave en GitHub

Ingresar en GitHub:

```text
Profile
→ Settings
→ SSH and GPG keys
→ New SSH key
```

Configurar:

```text
Title:
PC - Firma Git

Key type:
Signing Key

Key:
[Pegar aquí la clave pública]
```

Seleccionar **Signing Key**, ya que esta clave se utilizará para firmar commits.

---

## 5. Configurar Git para utilizar firmas SSH

Ejecutar:

```powershell
git config --global gpg.format ssh
```

Configurar la clave pública:

```powershell
git config --global user.signingkey "$HOME/.ssh/id_ed25519.pub"
```

Activar la firma automática de commits:

```powershell
git config --global commit.gpgsign true
```

La opción `--global` hace que esta configuración se aplique a los repositorios Git utilizados por este usuario de Windows.

---

## 6. Verificar la configuración

Ejecutar:

```powershell
git config --global --get gpg.format
git config --global --get user.signingkey
git config --global --get commit.gpgsign
```

El resultado debe ser similar a:

```text
ssh
C:\Users\USUARIO\.ssh\id_ed25519.pub
true
```

También se puede comprobar de dónde proviene cada configuración:

```powershell
git config --show-origin --get gpg.format
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
```

---

## 7. Crear un nuevo commit firmado

A partir de este momento el procedimiento normal de Git no cambia.

Modificar un archivo y ejecutar:

```powershell
git add .
git commit -m "docs: prueba de commit firmado"
```

Como:

```text
commit.gpgsign = true
```

Git intentará firmar automáticamente cada nuevo commit.

No es necesario utilizar `-S` en cada commit.

---

## 8. Comprobar que el commit contiene una firma

Ejecutar:

```powershell
git cat-file -p HEAD
```

Un commit firmado mediante SSH debe contener una sección similar a:

```text
gpgsig -----BEGIN SSH SIGNATURE-----
...
-----END SSH SIGNATURE-----
```

Si aparece `BEGIN SSH SIGNATURE`, el commit contiene una firma SSH.

---

## 9. Subir el commit a GitHub

Ejecutar:

```powershell
git push
```

Al consultar el commit en GitHub debería aparecer:

```text
Verified
```

Esto significa que GitHub pudo verificar criptográficamente la firma del commit mediante una clave de firma registrada en la cuenta.

> `Verified` no significa que GitHub haya revisado o aprobado el código.  
> Significa que la firma criptográfica del commit pudo ser verificada.

---

## 10. Si el último commit fue creado sin firma

Si todavía **NO se ha realizado `push`**, se puede reemplazar el último commit por una versión firmada:

```powershell
git commit --amend --no-edit -S
```

Donde:

```text
--amend    Reemplaza el último commit
--no-edit  Conserva el mensaje del commit
-S         Firma explícitamente el commit
```

El hash del commit cambiará.

Ejemplo:

```text
ANTES

e837a9f  docs: prueba de commit firmado
         SIN FIRMA


git commit --amend --no-edit -S


DESPUÉS

d1e210f  docs: prueba de commit firmado
         FIRMA SSH
```

No son dos commits en el historial final. El segundo **reemplaza** al primero.

Después comprobar:

```powershell
git cat-file -p HEAD
```

y finalmente:

```powershell
git push
```

---

## Flujo normal después de configurar la firma

La configuración se realiza una sola vez para el usuario de Windows.

Después el trabajo diario continúa normalmente:

```powershell
git add .
git commit -m "feat: nueva funcionalidad"
git push
```

El proceso internamente será:

```text
Modificar archivos
       ↓
    git add
       ↓
   git commit
       ↓
Firma SSH automática
       ↓
    git push
       ↓
     GitHub
       ↓
   Verified ✓
```

No es necesario volver a generar la clave SSH ni volver a configurar `commit.gpgsign` para cada repositorio.
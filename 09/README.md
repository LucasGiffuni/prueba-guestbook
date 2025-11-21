# Guestbook Simple con AWS Amplify

## Guestbook Serverless con Verificación SMS

Este proyecto crea un **Guestbook** (Muro de Publicaciones) simple utilizando AWS Amplify. Implementa autenticación obligatoria a través de **Amazon Cognito** con **verificación por SMS** y utiliza una API GraphQL (AppSync/DynamoDB) para almacenar los mensajes, asegurando que cada usuario solo pueda ver y **borrar sus propias publicaciones**.

![diagrama](amp.svg)

---

### Pre-requisitos

1.  Una cuenta de **AWS** activa.
2.  **Node.js** y **npm** instalados.
3.  **AWS Amplify CLI** instalado globalmente:
    ```bash
    npm install -g @aws-amplify/cli
    ```
4.  La Amplify CLI configurada con tus credenciales de AWS:
    ```bash
    amplify configure
    ```

---

### Pasos de Configuración y Despliegue

#### 1. Clonar el Repositorio e Inicializar Amplify

```bash
# Crea un directorio para tu proyecto y entra en él
# Esto ya lo hiciste en el paso 1
# cd amplify-guestbook

# Inicializa el proyecto de Amplify
amplify init
# Sigue las instrucciones:
# - project name: (guestbook-simple)
# - environment name: (dev)
# - default editor: (tu elección)
# - Choose the type of app: javascript
# - framework: none
# - source directory path: src
# - distribution directory path: dist
# - build command: npm run build
# - start command: npm run start
```

#### 2. Configurar Autenticación (Cognito con SMS)

Añade la categoría de autenticación y configúrala para usar el número de teléfono.

```bash
amplify add auth
```

**Sigue las indicaciones (opciones sugeridas):**
* **Do you want to use the default authentication and security configuration?** `Default configuration`
* **How do you want users to be able to sign in?** `Phone Number` (Esto activa la verificación por SMS)
* **Do you want to configure advanced settings?** `No`

#### 3. Configurar la API (GraphQL)

Añade la categoría de la API (AppSync y DynamoDB).

```bash
amplify add api
```

**Sigue las indicaciones (opciones sugeridas):**
* **Please select from one of the below mentioned services:** `GraphQL`
* **Choose the authorization type for the API:** `Amazon Cognito User Pool`
* **Do you have an annotated GraphQL schema?** `No`
* **Do you want a guided schema creation?** `Yes`
* **What best describes your project:** `Single object with fields`
* **Do you want to edit the schema now?** `Yes`

**Esquema GraphQL (`amplify/backend/api/guestbooksimple/schema.graphql`):**

Reemplaza el contenido del archivo con el siguiente código.

```graphql
type Message @model @auth(rules: [
  # Permite que el dueño (owner) cree, lea, actualice y borre.
  { allow: owner, operations: [create, read, update, delete] },
  # Permite que TODOS los usuarios autenticados (public) puedan leer la lista de todos los mensajes.
  { allow: public, provider: userPools, operations: [read] }
]) {
  id: ID!
  content: String!
  createdAt: AWSDateTime!
}
```

#### 4. Desplegar el Backend

Sube todas las configuraciones a AWS.

```bash
amplify push
```

#### 5. Copiar Configuración

Copia el archivo de configuración generado por Amplify al directorio `src`.

```bash
cp aws-exports.js src/amplify-config.js
```

---

## 6. Ejecutar y Probar

Simplemente abre el archivo `src/index.html` en tu navegador para probarlo localmente.

---

## 7. Publicar la Aplicación con Amplify Hosting

Una vez que tu backend (Cognito y GraphQL API) está configurado con `amplify push`, el último paso es desplegar el frontend estático (`src/index.html` y `src/app.js`) usando **Amplify Hosting** para obtener una URL pública.

### 7.1. Inicializar y Configurar Hosting

1.  **Añade la categoría de Hosting:**
    ```bash
    amplify add hosting
    ```
2.  **Elige las opciones sugeridas:**
    * **Select the plugin module to execute:** `Amplify Hosting`
    * **Choose a type:** `Manual deployment` (Sencillo para archivos planos sin CI/CD).

### 7.2. Despliegue de Archivos

1.  **Asegúrate de que la configuración sea correcta:** El despliegue manual subirá el contenido del directorio que definiste al inicio (`src` en este caso).

2.  **Ejecuta el comando de publicación:**
    ```bash
    amplify publish
    ```

3.  **Resultado:** Amplify subirá tus archivos y la CLI te mostrará la **URL del sitio web publicado** (ejemplo: `https://master.xxxxxxx.amplifyapp.com`).

---

## 8. (Opcional) Despliegue Continuo con Git

Si deseas que la aplicación se actualice automáticamente cada vez que haces *push* a tu repositorio Git, sigue estos pasos *después* de configurar el hosting:

1.  **Inicializa Git y sube tu código:**
    ```bash
    git init
    # Asegúrate de haber agregado y commiteado todo antes de este paso
    git remote add origin <URL_DE_TU_REPO>
    git branch -M main
    git push -u origin main
    ```
2.  **Conéctate en la Consola AWS:** Ve a la **Consola de AWS Amplify** y selecciona **"Connect app"**.
3.  **Elige el repositorio y rama** (`main`).
4.  **Configuración de Build:** Asegúrate de que el `baseDirectory` en la configuración de *build* apunte a **`/src`** para que Amplify encuentre tu `index.html`.

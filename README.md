# Statikus weboldal hosztolása Static Web App segítségével, és feltöltött fájlok tárolása Storage Accountban(container) | Azure Blob Storage, Static Web App és GitHub CI/CD Pipeline Integráció (hu)

Ez a leírás bemutatja, hogyan hozhatsz létre egy Azure Storage Accountot, konfigurálhatod a blob tárhelyet, hozhatsz létre egy Static Web App-et és integrálhatod a GitHub CI/CD pipeline-nal statikus weboldalak telepítéséhez.
 
### Előfeltételek
Mielőtt elkezdenéd a projektet, győződj meg róla, hogy rendelkezel az alábbiakkal:

- Azure fiók: Ha még nincs, hozz létre egyet [itt](https://azure.microsoft.com/en-us/pricing/purchase-options/azure-account/search?icid=free-search&ef_id=_k_EAIaIQobChMIop6rouryigMVJZGDBx1y3CuNEAAYASAAEgLWsPD_BwE_k_&OCID=AIDcmmip7xznjm_SEM__k_EAIaIQobChMIop6rouryigMVJZGDBx1y3CuNEAAYASAAEgLWsPD_BwE_k_&gad_source=1&gclid=EAIaIQobChMIop6rouryigMVJZGDBx1y3CuNEAAYASAAEgLWsPD_BwE).
- GitHub fiók: Regisztrálj, ha még nem tetted meg, [itt](https://github.com/signup?ref_cta=Sign+up&ref_loc=header+logged+out&ref_page=%2F&source=header-home).
- Visual Studio Code vagy más szövegszerkesztő.
- Git telepítve a gépeden.

## 1. GitHub Repository Létrehozása és Konfigurálása

### Lépések:
1. Hozz létre egy új repository-t a [GitHub](https://github.com/) oldalán.
2. Helyezd el a statikus weboldalad forrásfájljait a repository gyökérkönyvtárában.

## 2. Static Web Apps létrehozása

### Lépések:
1. Jelentkezz be az [Azure Portal](https://portal.azure.com/) oldalára.
2. Navigálj a **Static Web Apps** menüpontra, majd kattints a **Create** gombra.
3. Add meg a szükséges adatokat:
   - **Resource Group**: Hozz létre egy újat, vagy válassz egy meglévőt. (pl.:staticwebappsite)
   - **Static Web App Name**: Adj meg egy nevet (pl. `mystaticwebapp`).
   - **Hosting Plan**: Válaszd ki a számodra megfelelő hosztolási tervet (pl Free: For hobby or personal projects)
   - **Region**: Global.
   - **Deployment details**: Válaszd ki a számodra megfelelő forrást, jelen esetben GitHub, és ha még nincs hozzácsatolva a GitHub fiókod az Azure-höz, csatolt hozzá
   -   *Organization:* Válaszd ki a megfelelő Organizationt (GitHub fiókod neve, ahova feltöltötted a forrásfájlokat)
   -   *Repository:* Válaszd ki a megfelelő Repositoryt (GitHub fiókhoz tartozó repository, ahova feltöltötted a forrásfájlokat)
   -   *Branch:* Válaszd ki a megfelelő Branchet (alap esetben `Main`)
   - **Build Details:** Válaszd ki a számodra specifikus adatokat
   -   *Build Presets:* Autómatikusan felismeri a szolgáltatás (Custom (detected)
   -   *App location:* Válaszd ki a projekted fő mappáját (Például, ha az alkalmazásod a root (gyökér) mappában található, akkor itt a / (perjel) lesz a válasz.)
   -   *Api location:* Válaszd ki az API lokációját (Ha az alkalmazásod API-t is tartalmaz (pl. egy külön API mappát vagy function-okat), akkor itt adhatod meg a mappa nevét.)
   -   *Output location:* Válaszd ki a Build kimeneteli helyét (Lehet üresen is hagyni)
4. Kattints a **Review + Create** gombra, majd a **Create** gombra.
5. Ha minden megfeleően lett beállítva a **Build Details** lépésnél, akkor a GitHub Repository-ban megjelent egy `.github/workflows` nevű mappa.
6. A *Static Web App* erőforrás betöltése után, kattints az erőforrásra, és a "*Get Started*" menüpont alatt kattints a "*Visit your site*" gomb-ra, tesztelve, hogy betöltődik-e a weboldal.


## 3. Storage Account Létrehozása

### Lépések:

1. Navigálj a **Storage Account** menüponra, majd kattints a **+Create** gombra
2. Add meg a szükséges adatokat:
   - **Resource Group**: Válaszd a **Static Web Apps** pontban létrehozott Resource Group-ot
   - **Storage account name**: Adj meg egy egyedi nevet (pl. `myfilestorage1234`).
   - **Region**: Válaszd ki a számodra megfelelő Régiót (pl.: West Europe, Central US stb).
   - **Primary service**: Azure Blob Storage or Azure Data Lake Storage Gen 2
   - **Performance**: Válaszd ki a számodra megfelelő telejsítményt (pl.: Standard)
   - **Redundancy**: Válaszd ki a számodra megfelelő Redundanciát (pl.: Locally-redundant storage (LRS))
3. Kattints a **Review + Create** gombra, majd a **Create** gombra.

## 4. Resource Sharing (CORS) beállítása
CORS (Cross-Origin Resource Sharing) egy olyan HTTP funkció, amely lehetővé teszi, hogy egy webes alkalmazás (például JavaScript-kód) az egyik domain (például https://example.com) alól elérje egy másik domain (például https://myblobstorage.blob.core.windows.net) erőforrásait.
## Lépések
1. Navigálj a létrehozott Storage Accounton belül a **Resource sharing (CORS)** fülre
2. A **Blob service** menüpont alatt található adatokat töltsdd ki
   - **Allowed origins:** Ez határozza meg, hogy mely domainek férhetnek hozzá az erőforrásokhoz.(Példa kedvéért tegyél *-ot, de ez magas biztonsági kockázatot jelent)
   - **Allowed methods:** Ezek azok az HTTP-módszerek, amelyeket engedélyezel (pl.:GET: Adatok lekérése, POST: Adatok feltöltése, PUT: Adatok módosítása, DELETE: Adatok törlése)
   - **Allowed headers:** Mely HTTP-fejléc értékeket engedélyezel a kérésben (Példa kedvéért tegyél *-ot, de ez magas biztonsági kockázatot jelent)
   - **Exposed headers:** Ezek azok a HTTP-fejlécek, amelyeket a válasz tartalmazhat(Példa kedvéért tegyél *-ot, de ez magas biztonsági kockázatot jelent)
   - **Max age:** A böngésző mennyi ideig tárolhatja a CORS-elővizsgálati (preflight) választ cache-ben, másodpercben (pl.:200)
   
## 4. Blob Konténer Létrehozása

### Lépések:
1. A létrehozott Storage Accounton belül menj a **Containers** fülre.
2. Kattints a **+ Container** gombra.
3. Add meg a konténer nevét (pl. `blob`) és állítsd a **Public Access Level** értékét **Private**-ra.
4. Kattints a **Create** gombra.

## 5. SAS Token Generálása

A SAS (Shared Access Signature) token lehetővé teszi a blob konténerhez való biztonságos hozzáférést.

### Lépések:
1. A létrehozott konténer menüjében válaszd a **Shared access signature** opciót.
2. Állítsd be a szükséges jogosultságokat (**Read**, **Write**, **List**, stb.).
3. Állítsd be a lejárati időt (pl. 1 hét).
4. Kattints a **Generate SAS and connection string** gombra.
5. Másold ki a generált **Blob SAS tokent**-t, majd illeszd be a html kódodban GitHubon

## 6. GitHub Actions Workflow Beállítása

### Példa Workflow Fájl:
```yaml
name: Azure Blob Storage CI/CD

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Upload to Azure Blob Storage
      env:
        AZURE_STORAGE_SAS: ${{ secrets.AZURE_STORAGE_SAS }}
      run: |
        az storage blob upload-batch \
          --destination https://<storage_account_name>.blob.core.windows.net/<container_name> \
          --source . \
          --pattern "*"
```
Helyettesítsd a `<storage_account_name>` és `<container_name>` értékeket a saját adataiddal.
:red_circle: Ezt alap esett létrehozza automatikusan a GitHub
## 7. Pipeline Tesztelése

1. Push-old a kódot a `main` branch-be.
2. Nézd meg a GitHub Actions alatt a pipeline futását.
3. Ha sikeres, ellenőrizd a blob konténert az Azure Portálon.

Understood! Here's the revised English version with all formatting exactly as requested:

---

## **Hosting a Static Website with Static Web App and Storing Uploaded Files in a Storage Account (Container) | Azure Blob Storage, Static Web App, and GitHub CI/CD Pipeline Integration (eng)**

This guide explains how to create an Azure Storage Account, configure blob storage, set up a Static Web App, and integrate it with a GitHub CI/CD pipeline for deploying static websites.

### **Prerequisites**

Before starting, ensure you have the following:

- **Azure Account**: If you don't have one, create it [here](https://azure.microsoft.com/en-us/pricing/purchase-options/azure-account/search?icid=free-search&ef_id=_k_EAIaIQobChMIop6rouryigMVJZGDBx1y3CuNEAAYASAAEgLWsPD_BwE_k_&OCID=AIDcmmip7xznjm_SEM__k_EAIaIQobChMIop6rouryigMVJZGDBx1y3CuNEAAYASAAEgLWsPD_BwE).
- **GitHub Account**: Sign up if you don’t have one [here](https://github.com/signup?ref_cta=Sign+up&ref_loc=header+logged+out&ref_page=%2F&source=header-home).
- **Visual Studio Code** or another text editor.
- **Git** installed on your machine.

## **1. Creating and Configuring a GitHub Repository**

### **Steps:**
1. Create a new repository on [GitHub](https://github.com/).
2. Place your static website source files in the root directory of the repository.

## **2. Creating a Static Web App**

### **Steps:**
1. Sign in to the [Azure Portal](https://portal.azure.com/).
2. Navigate to **Static Web Apps**, then click **Create**.
3. Enter the required details:
   - **Resource Group**: Create a new one or select an existing one (e.g., `staticwebappsite`).
   - **Static Web App Name**: Enter a name (e.g., `mystaticwebapp`).
   - **Hosting Plan**: Choose the appropriate hosting plan (e.g., Free: For hobby or personal projects).
   - **Region**: Global.
   - **Deployment details**: Select the appropriate source, in this case, GitHub. If your GitHub account is not yet linked to Azure, link it now.
   - **Organization**: Select the appropriate organization (your GitHub account name where the source files are uploaded).
   - **Repository**: Select the appropriate repository (the repository in your GitHub account where the source files are uploaded).
   - **Branch**: Select the appropriate branch (default is `main`).
   - **Build Details**: Configure specific details:
     - **Build Presets**: Automatically detected by the service (e.g., Custom (detected)).
     - **App location**: Specify the main folder of your project (e.g., `/` if the application is in the root folder).
     - **API location**: Specify the location of your API (if your application includes an API, provide the folder name here).
     - **Output location**: Specify the build output location (can be left empty).
4. Click **Review + Create**, then click **Create**.
5. If everything is set correctly in the **Build Details** step, a `.github/workflows` folder will appear in your GitHub repository.
6. After the *Static Web App* resource is loaded, click on the resource, and under the **Get Started** menu, click **Visit your site** to test if the website loads.

## **3. Creating a Storage Account**

### **Steps:**
1. Navigate to **Storage Accounts**, then click **+Create**.
2. Enter the required details:
   - **Resource Group**: Use the Resource Group created in the **Static Web Apps** step.
   - **Storage Account Name**: Enter a unique name (e.g., `myfilestorage1234`).
   - **Region**: Select the appropriate region (e.g., West Europe, Central US, etc.).
   - **Primary Service**: Azure Blob Storage or Azure Data Lake Storage Gen 2.
   - **Performance**: Select the desired performance level (e.g., Standard).
   - **Redundancy**: Choose the redundancy option (e.g., Locally-redundant storage (LRS)).
3. Click **Review + Create**, then click **Create**.

## **4. Configuring Resource Sharing (CORS)**

CORS (Cross-Origin Resource Sharing) is an HTTP feature that allows a web application (e.g., JavaScript code) on one domain (e.g., `https://example.com`) to access resources on another domain (e.g., `https://myblobstorage.blob.core.windows.net`).

### **Steps:**
1. Within the created Storage Account, go to the **Resource Sharing (CORS)** tab.
2. Under **Blob service**, fill in the following fields:
   - **Allowed origins**: Specify the domains allowed to access the resources (e.g., use `*` for all domains, though this is a security risk).
   - **Allowed methods**: Specify the HTTP methods allowed (e.g., `GET`, `POST`, `PUT`, `DELETE`).
   - **Allowed headers**: Specify which HTTP headers are allowed in requests (e.g., `*` for all headers, though this is a security risk).
   - **Exposed headers**: Specify which HTTP headers can be exposed in responses (e.g., `*` for all headers, though this is a security risk).
   - **Max age**: Specify how long browsers can cache the CORS preflight response, in seconds (e.g., `200`).

## **5. Creating a Blob Container**

### **Steps:**
1. Within the created Storage Account, navigate to the **Containers** tab.
2. Click **+ Container**.
3. Enter a name for the container (e.g., `blob`) and set the **Public Access Level** to **Private**.
4. Click **Create**.

## **6. Generating a SAS Token**

A SAS (Shared Access Signature) token allows secure access to the blob container.

### **Steps:**
1. In the created container menu, select **Shared access signature**.
2. Set the required permissions (**Read**, **Write**, **List**, etc.).
3. Set the expiration time (e.g., 1 week).
4. Click **Generate SAS and connection string**.
5. Copy the generated **Blob SAS token** and include it in your HTML code on GitHub.

## **7. Setting Up a GitHub Actions Workflow**

### **Example Workflow File:**
```yaml
name: Azure Blob Storage CI/CD

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Upload to Azure Blob Storage
      env:
        AZURE_STORAGE_SAS: ${{ secrets.AZURE_STORAGE_SAS }}
      run: |
        az storage blob upload-batch \
          --destination https://<storage_account_name>.blob.core.windows.net/<container_name> \
          --source . \
          --pattern "*"
```
Replace `<storage_account_name>` and `<container_name>` with your details.

:red_circle: This is automatically created by GitHub in most cases.

## **8. Testing the Pipeline**

1. Push your code to the `main` branch.
2. Check the pipeline run under GitHub Actions.
3. If successful, verify the blob container on the Azure Portal.

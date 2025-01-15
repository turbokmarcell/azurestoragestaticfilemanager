# Statikus weboldal hosztolása Static Web App segítségével, és feltöltött fájlok tárolása Storage Accountban(container) | Azure Blob Storage, Static Web App és GitHub CI/CD Pipeline Integráció

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

###Lépések###

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

## 7. Pipeline Tesztelése

1. Push-old a kódot a `main` branch-be.
2. Nézd meg a GitHub Actions alatt a pipeline futását.
3. Ha sikeres, ellenőrizd a blob konténert az Azure Portálon.


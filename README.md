# Gonzalo Porta · QA Technical Lead

Portfolio personal publicado en **[gportabopp.github.io](https://gportabopp.github.io)**.

QA Technical Lead en Buenos Aires, con foco en automatización web, mobile (iOS / Android) y backend, y en arquitectura de soluciones de testing reutilizables.

[LinkedIn](https://www.linkedin.com/in/gonzalo-porta-b53343184/) · [CV en español](cv/Gonzalo_Porta_CV_ES.pdf) · [CV in English](cv/Gonzalo_Porta_CV_EN.pdf)

---

## 🏗️ Arquitectura de soluciones de QA

Diseñé y mantengo **`commons_qa`**, una librería Python interna que funciona como capa común entre todos los frameworks de automatización de QA de una billetera digital. El código es privado; estos diagramas muestran la arquitectura en términos generales, sin datos internos.

### Diagrama transversal

Cada repo de automatización corre sus tests desde su propio pipeline y entra a `commons_qa` para preparar datos, hablar con los sistemas reales, reportar en Allure, validar sus merge requests con `qa-guard` y, desde un template de CI, detectar deploys nuevos de la app para disparar la regresión.

```mermaid
flowchart LR
    subgraph CI["CI/CD · pipeline de cada repo"]
        direction TB
        PIPE["tests · qa-guard · deploy"]
    end

    subgraph REPOS["Frameworks de automatización (10+ repos)"]
        direction TB
        MOB["📱 Mobile iOS / Android<br/>4 frameworks · Appium + pytest-bdd"]
        API["🔌 API / backend<br/>5 frameworks · Pytest + HTTP"]
        WEB["🌐 Web<br/>2 frameworks · Playwright"]
        TPL["🧩 Templates<br/>proyectos nuevos ya conectados"]
    end

    subgraph CQ["commons_qa · librería común"]
        direction TB
        TK["toolkit<br/>operaciones de negocio para steps"]
        SV["services<br/>clientes HTTP / SDK sobre BaseService"]
        UT["utils<br/>AllureHelper · allure_plugin · Config"]
        CLI["CLIs<br/>qa-guard · check-new-deploy · setup-local-env"]
        TCI["🆕 Template CI<br/>.check_new_deploy"]
        TK --> SV
        TCI --> CLI
    end

    subgraph SYS["Sistemas reales"]
        direction TB
        APIS["APIs del producto · Keycloak"]
        DATA[("Bases de datos · Oracle")]
        GCP["Google Cloud<br/>BigQuery · Pub/Sub · GCS"]
        BS["BrowserStack<br/>dispositivos reales / virtuales"]
    end

    REP[("Allure Results<br/>reporte por ejecución")]

    PIPE --> MOB & API & WEB
    PIPE -. "qa-guard en el MR" .-> CLI
    MOB & API & WEB & TPL --> TK
    MOB & API & WEB --> UT
    WEB & TPL -. "include" .-> TCI
    SV --> APIS & DATA & GCP & BS
    UT --> REP

    classDef core fill:#1f6feb,stroke:#0b3d91,color:#fff
    classDef new fill:#e8590c,stroke:#a83f06,color:#fff
    class TK,SV,UT,CLI core
    class TCI new
```

### 🆕 Detección automática de deploys (`check-new-deploy`)

Los repos de QA incluyen un template de CI de `commons_qa` y, desde un pipeline programado, detectan cuándo la app tiene un deploy nuevo en el ambiente de testing. Si lo hay, disparan la regresión sin esperar a que el equipo de infraestructura avise.

```mermaid
flowchart LR
    SCH["⏱️ Schedule del repo QA<br/>CHECK_DEPLOY=true"] --> JOB["job extends<br/>.check_new_deploy"]
    JOB --> CND["check-new-deploy<br/>(commons_qa)"]
    CND --> Q{"¿Deploy exitoso nuevo<br/>de la app en testing?"}
    Q -- "no" --> OK0["sin cambios<br/>espera al próximo schedule"]
    Q -- "sí" --> RUN["crea pipeline de regresión<br/>RUN_REGRESSION=true"]
    RUN --> UPD["actualiza<br/>LAST_TESTED_DEPLOY"]

    classDef new fill:#e8590c,stroke:#a83f06,color:#fff
    class JOB,CND new
```

### Ejemplo de punta a punta: framework mobile

Así recorre la arquitectura un test mobile, desde el escenario Gherkin hasta la app instalada en un dispositivo de BrowserStack. Los demás frameworks repiten el patrón: cambia la capa de ejecución (Appium, Playwright o HTTP), pero el ingreso a `commons_qa` para fixtures, reportes y gobierno es el mismo.

```mermaid
flowchart LR
    CIP["CI/CD pipeline"] -- "dispara pytest (-m marker)" --> CF["conftest.py<br/>fixtures driver / page / context"]
    CIP -. "qa-guard: análisis estático del MR" .-> CQ["commons_qa"]

    subgraph RUN["Entorno de ejecución · local o runner de CI"]
        direction LR
        FEAT["Feature files<br/>Gherkin BDD"] --> STEPS["Step definitions<br/>pytest-bdd"]
        CF --> STEPS
        STEPS --> PO["Page Objects<br/>BasePage + módulos"]
        STEPS --> HLP["Helpers<br/>pre / post-conditions"]
        PO --> DRV["DriverManager<br/>driver Appium único"]
        CF -- "AllureHelper · allure_plugin" --> CQ
        CQ --> AR[("Allure Results")]
    end

    subgraph CLOUD["BrowserStack · nube"]
        direction LR
        BSH["Appium Hub<br/>Device Cloud"] --> APP["App bajo prueba<br/>Android / iOS"]
    end

    DRV -- "Remote WebDriver" --> BSH

    classDef core fill:#1f6feb,stroke:#0b3d91,color:#fff
    class CQ core
```

---

## 📁 Contenido del repo

| Archivo | Qué es |
|---|---|
| `index.html` | El portfolio (una sola página, en español e inglés) |
| `cv/` | CV en PDF, en español e inglés |
| `img/` | Foto del portfolio |

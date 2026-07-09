# 🚀 Proyecto DevOps - Sistema de Ventas y Despachos en AWS EKS

<div align="center">
  <img src="https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white" />
  <img src="https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white" />
  <img src="https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white" />
  <img src="https://img.shields.io/badge/github%20actions-%232671E5.svg?style=for-the-badge&logo=githubactions&logoColor=white" />
  <img src="https://img.shields.io/badge/spring-%236DB33F.svg?style=for-the-badge&logo=spring&logoColor=white" />
  <img src="https://img.shields.io/badge/mysql-%234479A1.svg?style=for-the-badge&logo=mysql&logoColor=white" />
</div>

<br>

Este repositorio contiene el código fuente, la infraestructura como código (IaC), los manifiestos de despliegue y el pipeline de Integración y Entrega Continua (CI/CD) para una arquitectura de microservicios alojada de forma escalable y resiliente en **Amazon Elastic Kubernetes Service (EKS)**.

## 📌 Características Principales

* **Despliegue Automatizado:** Integración nativa con GitHub Actions para compilar, empaquetar y desplegar sin intervención manual.
* **Arquitectura Orientada a Microservicios:** Separación lógica de responsabilidades entre Ventas y Despachos para permitir escalabilidad independiente.
* **Alta Disponibilidad:** Balanceo de carga gestionado por AWS (ELB) hacia los pods de Kubernetes.
* **Seguridad y Aislamiento:** Base de datos restringida a la red interna del clúster (ClusterIP), inaccesible desde el exterior.

---

## 🛠️ Stack Tecnológico

| Categoría | Tecnología Utilizada |
| :--- | :--- |
| **Backend** | Java 17, Spring Boot, Maven |
| **Frontend** | HTML/CSS/JS (Empaquetado en Nginx/Apache) |
| **Base de Datos** | MySQL 8.0 |
| **Contenedores** | Docker, Docker Hub |
| **Orquestación** | Kubernetes (K8s) |
| **Infraestructura Cloud**| AWS (EKS, EC2, Elastic Load Balancer, CloudShell) |
| **CI/CD** | GitHub Actions |

---

## 📂 Estructura del repositorio

├── .github/
│   └── workflows/
│       └── deploy.yml          # Pipeline de GitHub Actions
├── k8s/                        # Manifiestos de Kubernetes (.yaml)
│   ├── 01-mysql.yaml           # Deployment, Secret y Service de Base de Datos
│   ├── 02-back-ventas.yaml     # Deployment y Service (Backend Ventas)
│   ├── 03-back-despachos.yaml  # Deployment y Service (Backend Despachos)
│   └── 04-frontend.yaml        # Deployment y LoadBalancer (Frontend)
├── cluster.yaml                # Archivo IaC para aprovisionamiento del clúster
└── README.md                   # Documentación general

---

## ⚙️ Pipeline CI/CD (GitHub Actions)

El proyecto cuenta con un flujo de trabajo totalmente automatizado. Cada vez que se realiza un push a la rama main, se activan los siguientes Jobs:

Build & Test: Compila el código fuente de Java utilizando Maven.

Dockerization: Construye las imágenes Docker para el frontend y los backends, publicándolas en Docker Hub con la etiqueta latest.

Deploy to AWS: Se autentica en la cuenta de AWS Academy, actualiza el contexto de kubeconfig y aplica automáticamente los archivos .yaml de la carpeta k8s/ en el clúster EKS.

---

## 🏗️ Arquitectura del Proyecto

El sistema se ejecuta dentro del Namespace `tienda` en Kubernetes, dividiéndose en los siguientes componentes:

1. **Frontend (Despacho Dashboard):** Interfaz gráfica expuesta a internet a través de un servicio `LoadBalancer` de AWS.
2. **Microservicio Back-Ventas:** API encargada de procesar la lógica de ventas y carrito de compras.
3. **Microservicio Back-Despachos:** API encargada de gestionar la logística, tracking y envíos.
4. **Base de Datos (MySQL):** Motor relacional desplegado internamente, accesible exclusivamente por los microservicios mediante resolución DNS de Kubernetes.

### Diagrama de Flujo e Infraestructura

```mermaid
flowchart LR
    %% ACTORES EXTERNOS
    User(["👤 Usuario Final"])
    Hub[("🐳 Docker Hub")]

    %% PIPELINE CI/CD
    subgraph CI ["GitHub Actions (Pipeline CI/CD)"]
        direction TB
        Code["📝 Código (Push)"]
        Build["⚙️ Maven Build"]
        Img["📦 Docker Build"]
        Deploy["🚀 Deploy EKS"]

        Code --> Build --> Img --> Deploy
    end

    %% INFRAESTRUCTURA AWS
    subgraph AWS ["☁️ AWS EKS Cluster"]
        direction TB
        ELB["🌐 Elastic Load Balancer (Puerto 80)"]

        subgraph K8S ["Namespace: tienda"]
            direction TB
            Front["🖥️ frontend (Pod: 80)"]
            Ventas["🛒 back-ventas (8080)"]
            Despachos["🚚 back-despachos (8080)"]
            SVC["🔌 mysql-service"]
            DB[("🗄️ mysql-db (3306)")]

            Front --> Ventas
            Front --> Despachos
            Ventas --> SVC
            Despachos --> SVC
            SVC ==> DB
        end
        ELB ===> Front
    end

    %% RELACIONES Y FLUJOS
    User ===>|Acceso HTTP| ELB
    Img -.->|Publica Imagen| Hub
    Deploy -.->|kubectl apply| K8S
    
    Hub -.->|Descarga Imagen| Front
    Hub -.->|Descarga Imagen| Ventas
    Hub -.->|Descarga Imagen| Despachos

    %% ASIGNACION DE ESTILOS
    class User user;
    class Hub docker;
    class Code,Build,Img,Deploy github;
    class ELB aws;
    class Front,SVC k8s;
    class Ventas,Despachos spring;
    class DB db;

    %% DEFINICION DE COLORES
    classDef user fill:#8e44ad,stroke:#fff,stroke-width:2px,color:#fff,font-weight:bold,rx:10,ry:10;
    classDef github fill:#24292e,stroke:#fff,stroke-width:2px,color:#fff,font-weight:bold,rx:5,ry:5;
    classDef aws fill:#ff9900,stroke:#232f3e,stroke-width:2px,color:#232f3e,font-weight:bold,rx:5,ry:5;
    classDef k8s fill:#326ce5,stroke:#fff,stroke-width:2px,color:#fff,font-weight:bold,rx:5,ry:5;
    classDef spring fill:#6db33f,stroke:#fff,stroke-width:2px,color:#fff,font-weight:bold,rx:5,ry:5;
    classDef db fill:#00758f,stroke:#fff,stroke-width:2px,color:#fff,font-weight:bold,rx:10,ry:10;
    classDef docker fill:#0db7ed,stroke:#fff,stroke-width:2px,color:#fff,font-weight:bold,rx:10,ry:10;

    style CI fill:#f6f8fa,stroke:#d1d5da,stroke-width:2px,stroke-dasharray: 5 5,rx:10,ry:10;
    style AWS fill:#f8f9fa,stroke:#ff9900,stroke-width:2px,rx:10,ry:10;
    style K8S fill:#e6f0fa,stroke:#326ce5,stroke-width:2px,rx:10,ry:10;

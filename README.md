# 🚀 Proyecto DevOps - Sistema de Ventas y Despachos en AWS EKS

<div align="center">
  <img src="https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white" />
  <img src="https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white" />
  <img src="https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white" />
  <img src="https://img.shields.io/badge/github%20actions-%232671E5.svg?style=for-the-badge&logo=githubactions&logoColor=white" />
  <img src="https://img.shields.io/badge/spring-%236DB33F.svg?style=for-the-badge&logo=spring&logoColor=white" />
</div>

<br>

Este repositorio contiene el código fuente, la infraestructura como código (IaC), los manifiestos de despliegue y el pipeline de Integración y Entrega Continua (CI/CD) para una arquitectura de microservicios alojada en **Amazon Elastic Kubernetes Service (EKS)**.

## 🏗️ Arquitectura del Proyecto

El sistema está compuesto por cuatro componentes principales que se ejecutan dentro del Namespace `tienda` en Kubernetes:

1. **Frontend (Despacho Dashboard):** Interfaz gráfica expuesta a internet a través de un `LoadBalancer` de AWS.
2. **Microservicio Back-Ventas:** API desarrollada en Spring Boot (Java) encargada de la lógica de ventas.
3. **Microservicio Back-Despachos:** API desarrollada en Spring Boot encargada de gestionar los envíos.
4. **Base de Datos (MySQL):** Motor relacional MySQL 8.0 aislado del exterior, accesible solo por los microservicios a través del clúster interno.

### Diagrama de Flujo e Infraestructura

```mermaid
flowchart TD
    %% 1. DEFINICION DE ACTORES Y COMPONENTES
    U["Usuario Final"]
    Hub["Docker Hub"]

    subgraph CI_CD ["GitHub Actions (CI/CD Pipeline)"]
        direction LR
        Code["Codigo (Push)"]
        Build["Maven Build & Test"]
        Img["Docker Build & Push"]
        Deploy["EKS Deploy"]

        Code ==> Build ==> Img ==> Deploy
    end

    subgraph AWS ["Amazon Web Services (EKS Cluster)"]
        direction TB
        ELB["Elastic Load Balancer (Puerto 80)"]

        subgraph Namespace ["Kubernetes Namespace: tienda"]
            direction TB
            Front["tienda-frontend (Pod: 80)"]
            Ventas["back-ventas (Spring Boot: 8080)"]
            Despachos["back-despachos (Spring Boot: 8080)"]
            SVC["mysql-service (ClusterIP: 3306)"]
            MySQL[("mysql-db (Base de Datos)")]

            Front --> Ventas
            Front --> Despachos
            Ventas --> SVC
            Despachos --> SVC
            SVC === MySQL
        end
    end

    %% 2. CONEXIONES GLOBALES
    U === ELB
    ELB === Front
    
    Img -.-> Hub
    Hub -.-> Front
    Hub -.-> Ventas
    Hub -.-> Despachos
    Deploy -.-> Namespace

    %% 3. DEFINICION DE COLORES (Clases CSS)
    classDef user fill:#8e44ad,stroke:#fff,stroke-width:2px,color:#fff,font-weight:bold,rx:10,ry:10;
    classDef github fill:#24292e,stroke:#fff,stroke-width:2px,color:#fff,font-weight:bold,rx:5,ry:5;
    classDef aws fill:#ff9900,stroke:#232f3e,stroke-width:2px,color:#232f3e,font-weight:bold,rx:5,ry:5;
    classDef k8s fill:#326ce5,stroke:#fff,stroke-width:2px,color:#fff,font-weight:bold,rx:5,ry:5;
    classDef spring fill:#6db33f,stroke:#fff,stroke-width:2px,color:#fff,font-weight:bold,rx:5,ry:5;
    classDef db fill:#00758f,stroke:#fff,stroke-width:2px,color:#fff,font-weight:bold,rx:10,ry:10;
    classDef docker fill:#0db7ed,stroke:#fff,stroke-width:2px,color:#fff,font-weight:bold,rx:10,ry:10;

    %% 4. ASIGNACION DE COLORES A LOS NODOS
    class U user;
    class Hub docker;
    class Code,Build,Img,Deploy github;
    class ELB aws;
    class Front,SVC k8s;
    class Ventas,Despachos spring;
    class MySQL db;

    %% 5. ESTILOS DE FONDOS PARA LOS CUADROS (Subgrafos)
    style CI_CD fill:#f6f8fa,stroke:#d1d5da,stroke-width:2px,stroke-dasharray: 5 5,rx:10,ry:10;
    style AWS fill:#f8f9fa,stroke:#ff9900,stroke-width:2px,rx:10,ry:10;
    style Namespace fill:#e6f0fa,stroke:#326ce5,stroke-width:2px,rx:10,ry:10;

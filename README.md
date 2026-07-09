# 🚀 Sistema de Ventas y Despachos - Orquestación en AWS EKS

<div align="center">
  <img src="https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=java&logoColor=white" />
  <img src="https://img.shields.io/badge/Spring_Boot-6DB33F?style=for-the-badge&logo=spring-boot&logoColor=white" />
  <img src="https://img.shields.io/badge/MySQL-00000F?style=for-the-badge&logo=mysql&logoColor=white" />
  <img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" />
  <img src="https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white" />
  <img src="https://img.shields.io/badge/Amazon_AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white" />
  <img src="https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white" />
</div>

<br>

Este repositorio contiene el código fuente, la infraestructura como código (IaC) y los manifiestos de Kubernetes para desplegar un sistema transaccional de microservicios. Todo el ciclo de vida, desde la compilación hasta el despliegue en la nube, está automatizado mediante **GitHub Actions** hacia un clúster de **Amazon EKS (Elastic Kubernetes Service)**.

---

## 📋 Tabla de Contenidos
1. [Descripción General](#-descripción-general)
2. [Arquitectura del Sistema](#-arquitectura-del-sistema)
3. [Estructura del Proyecto](#-estructura-del-proyecto)
4. [Flujo CI/CD (Pipeline)](#-flujo-cicd-pipeline)
5. [Despliegue e Infraestructura (AWS Academy)](#-despliegue-e-infraestructura-aws-academy)
6. [Instrucciones de Uso](#-instrucciones-de-uso)
7. [Limpieza de Recursos](#-limpieza-de-recursos)

---

## 📖 Descripción General
El sistema simula el ecosistema backend de una tienda en línea. Está dividido en microservicios independientes que interactúan entre sí dentro de un entorno aislado (Namespace) en Kubernetes. Proporciona una interfaz gráfica (Despacho Dashboard) accesible públicamente que consume las APIs internas para la gestión de compras y despachos logísticos.

---

## 🏛️ Arquitectura del Sistema

A continuación se detalla cómo interactúan los componentes dentro del clúster y cómo viaja el código desde este repositorio hasta AWS.

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

    %% 3. DEFINICION DE COLORES
    classDef user fill:#8e44ad,stroke:#fff,stroke-width:2px,color:#fff,font-weight:bold,rx:10,ry:10;
    classDef github fill:#24292e,stroke:#fff,stroke-width:2px,color:#fff,font-weight:bold,rx:5,ry:5;
    classDef aws fill:#ff9900,stroke:#232f3e,stroke-width:2px,color:#232f3e,font-weight:bold,rx:5,ry:5;
    classDef k8s fill:#326ce5,stroke:#fff,stroke-width:2px,color:#fff,font-weight:bold,rx:5,ry:5;
    classDef spring fill:#6db33f,stroke:#fff,stroke-width:2px,color:#fff,font-weight:bold,rx:5,ry:5;
    classDef db fill:#00758f,stroke:#fff,stroke-width:2px,color:#fff,font-weight:bold,rx:10,ry:10;
    classDef docker fill:#0db7ed,stroke:#fff,stroke-width:2px,color:#fff,font-weight:bold,rx:10,ry:10;

    %% 4. ASIGNACION DE COLORES
    class U user;
    class Hub docker;
    class Code,Build,Img,Deploy github;
    class ELB aws;
    class Front,SVC k8s;
    class Ventas,Despachos spring;
    class MySQL db;

    %% 5. ESTILOS DE FONDOS
    style CI_CD fill:#f6f8fa,stroke:#d1d5da,stroke-width:2px,stroke-dasharray: 5 5,rx:10,ry:10;
    style AWS fill:#f8f9fa,stroke:#ff9900,stroke-width:2px,rx:10,ry:10;
    style Namespace fill:#e6f0fa,stroke:#326ce5,stroke-width:2px,rx:10,ry:10;

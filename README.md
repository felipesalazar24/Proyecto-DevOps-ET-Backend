# 🚀 Proyecto DevOps - Sistema de Ventas y Despachos en AWS EKS

![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/github%20actions-%232671E5.svg?style=for-the-badge&logo=githubactions&logoColor=white)
![Spring Boot](https://img.shields.io/badge/spring-%236DB33F.svg?style=for-the-badge&logo=spring&logoColor=white)

Este repositorio contiene la infraestructura como código (IaC), los manifiestos de despliegue y el pipeline de Integración y Entrega Continua (CI/CD) para una arquitectura de microservicios alojada en **Amazon Elastic Kubernetes Service (EKS)**.

## 🏗️ Arquitectura del Proyecto

El sistema está compuesto por cuatro componentes principales que se ejecutan dentro del Namespace `tienda` en Kubernetes:

1. **Frontend (Despacho Dashboard):** Interfaz gráfica expuesta a internet a través de un `LoadBalancer` de AWS.
2. **Microservicio Back-Ventas:** API desarrollada en Spring Boot (Java) encargada de la lógica de ventas.
3. **Microservicio Back-Despachos:** API desarrollada en Spring Boot encargada de gestionar los envíos.
4. **Base de Datos (MySQL):** Motor relacional MySQL 8.0 aislado del exterior, accesible solo por los microservicios a través del clúster interno.

## ⚙️ Pipeline CI/CD (GitHub Actions)

El proyecto cuenta con un flujo de trabajo totalmente automatizado. Cada vez que se realiza un `push` a la rama `main`, se activan los siguientes *Jobs*:

- **Build & Test:** Compila el código fuente de Java utilizando Maven.
- **Dockerization:** Construye las imágenes Docker para el frontend y los backends, publicándolas en Docker Hub.
- **Deploy to AWS:** Se autentica en AWS Academy, actualiza el `kubeconfig` y aplica automáticamente los archivos `.yaml` de la carpeta `/k8s` en el clúster EKS.

## 🛠️ Requisitos Previos (Infraestructura)

Para que el pipeline funcione correctamente, se debe contar con un clúster EKS activo en AWS. Debido a las restricciones de roles IAM en **AWS Academy / Learner Lab**, el clúster fue levantado utilizando el siguiente archivo `cluster.yaml` con `eksctl`, el cual reutiliza el `LabRole` por defecto:

```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: devopseks
  region: us-east-1

iam:
  serviceRoleARN: "arn:aws:iam::831940482694:role/LabRole"

managedNodeGroups:
  - name: nodos-tienda
    instanceType: t3.medium
    minSize: 2
    maxSize: 2
    iam:
      instanceRoleARN: "arn:aws:iam::831940482694:role/LabRole"

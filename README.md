##  Proyecto Híbrido Big-Data Serverless: GPU, Spark & Actor Model

Este repositorio contiene la implementación completa de un **Sistema Híbrido de Procesamiento Big Data en Entornos Serverless**, diseñado para combinar la potencia de cómputo heterogéneo en GPU/CPU (CUDA & OpenMP), el procesamiento distribuido en masa (Apache Spark) y la orquestación asíncrona reactiva mediante el Modelo de Actores (Pykka / Akka Pattern).

---

## 📌 Tabla de Contenidos
1. [Descripción General](#-descripción-general)
2. [Arquitectura del Sistema](#-arquitectura-del-sistema)
3. [Requisitos Previos e Instalación](#-requisitos-previos-e-instalación)
4. [Estructura del Repositorio](#-estructura-del-repositorio)
5. [Instrucciones de Compilación y Ejecución](#-instrucciones-de-compilación-y-ejecución)
6. [Resultados y Comparativa de Rendimiento](#-resultados-y-comparativa-de-rendimiento)
7. [Licencia y Créditos](#-licencia-y-créditos)

---

## 📖 Descripción General

El objetivo de esta solución es ofrecer un pipeline *serverless* altamente escalable y tolerante a fallos para procesar datasets numéricos a gran escala (1,000,000 de registros):
1. **Preprocesamiento en GPU/CPU:** Se implementa una librería compartida en `C++/CUDA` con pragmas `OpenMP` para ejecutar una normalización *Min-Max* masivamente paralela.
2. **Orquestación con Actores:** Un sistema de actores en Python (`Pykka`) valida los datos, delega la ejecución de la función de preprocesamiento, invoca el job de Spark de forma no bloqueante y consolida las respuestas.
3. **Procesamiento Distribuido en Spark:** Se comparan experimentalmente dos estrategias analíticas sobre **PySpark**: la API de nivel bajo **RDD (Resilient Distributed Datasets)** frente a la API de nivel alto **DataFrame** (potenciada por Catalyst Optimizer y Tungsten Engine).

---

## 🏗️ Arquitectura del Sistema

```text
                        +---------------------------------------+
                        |  Cliente HTTP / API Event Request    |
                        +---------------------------------------+
                                            |
                                            v
                        +---------------------------------------+
                        |   JobManagerActor (Orquestador)       |
                        +---------------------------------------+
                                            |
           +--------------------------------+--------------------------------+
           |                                |                                |
           v                                v                                v
+----------------------+        +----------------------+        +----------------------+
|   ValidationActor    |        |    GPULambdaActor    |        |  SparkLauncherActor  |
|  (Validación Input)  |        | (CUDA/C++ .so Shared) |        |  (PySpark RDD & DF)  |
+----------------------+        +----------------------+        +----------------------+
                                            |                                |
                                            v                                v
                                   [ dataset_gpu.csv ] --------------> [ Spark Metrics ]

```
## 🔧 Requisitos Previos e Instalación

Entorno Sugerido
* Google Colab (con GPU Tesla T4) o Servidor Ubuntu Linux 20.04/22.04 LTS.
* Python: 3.10+ / 3.12+
* Java: OpenJDK 11 (Requerido por Apache Spark)
* NVIDIA CUDA Toolkit: nvcc y controladores NVIDIA instalados.

#### Dependencias de Software
Para instalar las bibliotecas necesarias, ejecute:

```text

# Actualización del sistema e instalación de OpenJDK 11
sudo apt-get update -qq
sudo apt-get install -y openjdk-11-jdk-headless build-essential g++

# Instalación de librerías Python necesarias
pip install --upgrade pyspark pykka numpy
```

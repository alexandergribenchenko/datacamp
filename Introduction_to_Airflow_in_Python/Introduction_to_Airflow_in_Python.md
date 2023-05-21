# Introduction to Airflow in Python

# Chapter 01. Intro to Airflow.

Contenido ejemplo de un archivo `ejemplo.sh`:
```bash
#!/bin/bash
# Crear la carpeta en la misma ubicación donde se ejecuta el script
mkdir "./carpeta_nueva"
```

Contenido ejemplo de un archivo `ejemplo.sh`:
```bash
#!/bin/bash

# Crear la carpeta en la misma ubicación donde se ejecuta el script
mkdir "./carpeta_01"
cd "./carpeta_01"

# Descargar archivos desde repositorios en la red
wget https://data.sba.gov/dataset/8aa276e2-6cab-4f86-aca4-a7dde42adf24/resource/aab8e9f9-36d1-42e1-b3ba-e59c79f1d7f0/download/ppp-data-dictionary.xlsx
wget https://data.sba.gov/dataset/8aa276e2-6cab-4f86-aca4-a7dde42adf24/resource/137436c9-408e-47e9-a7f3-b9a1871c4e11/download/public_up_to_150k_1_230331.csv
wget https://data.sba.gov/dataset/8aa276e2-6cab-4f86-aca4-a7dde42adf24/resource/67b6b208-7116-4e8d-9a56-7168b42cda4a/download/public_up_to_150k_2_230331.csv
```



Para ejecutar un archivo `.sh` debemos ejecutar en un terminal de linux (por ejemplo, git bash si estamos en Windows) :
```bash
bash ejemplo.sh
```

# Chapter 02. Implementing Airflow DAGs.

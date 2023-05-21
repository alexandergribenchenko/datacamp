# Introduction to Airflow in Python

# Chapter 01. Intro to Airflow.

## 01. Airflow: comandos básicos
-  `airflow` : Me muestra todos los posibles subcomandos:  `backfill,list_dag_runs,list_tasks,clear,pause,unpause,trigger_dag,delete_dag,pool,variables,kerberos,render,run,initdb,list_dags,dag_state,task_failed_deps,task_state,serve_logs,test,webserver,resetdb,upgradedb,scheduler,worker,flower,version,connections,create_user,delete_user,list_users,sync_perm,next_execution,rotate_fernet_key`
-  `airflow version`: me permite saber la version de Airflow instalada.
-  `airflow -h`: me permite obtener descripciones de todos los subcomandos posibles a ejecutar.
-  `airflow list_dags`: me entrega una lista de todos los dags dentro de airflow.
-  `airflow webserver -p 9090` o `airflow webserver --port 9090`: despliega la interfaz web de airflow en el puerto 9090.

## 02. Airflow: instanciar un DAG básico
```python
# Import the DAG object
from airflow.models import DAG

# Define the default_args dictionary
default_args = {
  'owner': 'dsmith',
  'start_date': datetime(2020, 1, 14),
  'retries': 2
}

# Instantiate the DAG object
etl_dag = DAG(dag_id='example_etl', default_args=default_args)
```

## 03. Airflow: BashOperator examples
```python
from airflow.operators.bash_operator import BashOperator

example_task = BashOperator(task_id='bash_ex',
bash_command='echo 1',
dag=dag_al_que_quiero_asignar)

bash_task = BashOperator(task_id='clean_addresses',
bash_command='cat addresses.txt | awk "NF==10" > cleaned.txt',
dag=dag_al_que_quiero_asignar)

bash_task_sh = BashOperator(
task_id='bash_script_example',
bash_command='runcleanup.sh',
dag=ml_dag)
```

## 04. Airflow: Orden tareas

### 04.01. Ejemplo 01.
```python
from airflow.models import DAG
from airflow.operators.bash_operator import BashOperator
from datetime import datetime

default_args = {
  'owner': 'dsmith',
  'start_date': datetime(2020, 2, 12),
  'retries': 1
}

codependency_dag = DAG('codependency', default_args=default_args)

task1 = BashOperator(task_id='first_task',
                     bash_command='echo 1',
                     dag=codependency_dag)

task2 = BashOperator(task_id='second_task',
                     bash_command='echo 2',
                     dag=codependency_dag)

task3 = BashOperator(task_id='third_task',
                     bash_command='echo 3',
                     dag=codependency_dag)

# task1 must run before task2 which must run before task3
task1 >> task2
task2 >> task3
```

### 04.01. Ejemplo 02.
```python
# Import the BashOperator
from airflow.operators.bash_operator import BashOperator

# Define the BashOperator 
cleanup = BashOperator(
    task_id='cleanup_task',
    # Define the bash_command
    bash_command='cleanup.sh',
    # Add the task to the dag
    dag=analytics_dag
)

# Define a second operator to run the `consolidate_data.sh` script
consolidate = BashOperator(
    task_id='consolidate_task',
    bash_command='consolidate_data.sh',
    dag=analytics_dag)

# Define a final operator to execute the `push_data.sh` script
push_data = BashOperator(
    task_id='pushdata_task',
    bash_command='push_data.sh',
    dag=analytics_dag)


# Define a new pull_sales task
pull_sales = BashOperator(
    task_id='pullsales_task',
    bash_command='wget https://salestracking/latestinfo?json',
    dag=analytics_dag
)

# Set pull_sales to run prior to cleanup
pull_sales >> cleanup

# Configure consolidate to run after cleanup
cleanup >> consolidate

# Set push_data to run last
consolidate >> push_data
```




## 05. Airflow: archivos `.sh`
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

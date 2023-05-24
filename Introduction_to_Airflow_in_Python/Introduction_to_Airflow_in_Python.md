# Introduction to Airflow in Python

# Chapter 01. Intro to Airflow.

## 01. Airflow: comandos básicos
-  `airflow` : Me muestra todos los posibles subcomandos:  `backfill,list_dag_runs,list_tasks,clear,pause,unpause,trigger_dag,delete_dag,pool,variables,kerberos,render,run,initdb,list_dags,dag_state,task_failed_deps,task_state,serve_logs,test,webserver,resetdb,upgradedb,scheduler,worker,flower,version,connections,create_user,delete_user,list_users,sync_perm,next_execution,rotate_fernet_key`
-  `airflow version`: me permite saber la version de Airflow instalada.
-  `airflow -h`: me permite obtener descripciones de todos los subcomandos posibles a ejecutar.
-  `airflow list_dags`: me entrega una lista de todos los dags dentro de airflow.
-  `airflow webserver -p 9090` o `airflow webserver --port 9090`: despliega la interfaz web de airflow en el puerto 9090.
- `head airflow/airflow.cfg`: nos permite visualizar el archivo de configuración de airflow, que casi simpre esta al interior de la carpeta `airflow`, y y con `head` podemos ver las primeras lineas entre las cuales podemos ver el `dags_folder`.


## 01. Airflow: Debugg de los DAGs
1. Verificar cual es la carpeta por defecto donde deben estar almacenados los dags, esto lo vemos en el `airflow.cfg
2. Verificar que el dag que queremos ejecutar se encuentra dentro de esa carpeta.
3. Ejecutar en consola el archivo `.py` que contiene el dag y verificar que se ejecute normalmente sin entregar ninguna salida, ni ningun tipo de error.
4. Entrar directamante al DAG ( a su archivo `.py`) y verificar que se encuentre bien configurado.



### 01.01. Comandos complementarios de bash
- `cat workspace/dags/codependent.py`: nos permite visualizar en consola un archivo determiando.


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

## 02. Airflow: instanciar un DAG con squedule

cron syntax to configure a schedule of every Wednesday at 12:30pm
```python
# Update the scheduling arguments as defined
default_args = {
  'owner': 'Engineering',
  'start_date': datetime(2019, 11, 1),
  'email': ['airflowresults@datacamp.com'],
  'email_on_failure': False,
  'email_on_retry': False,
  'retries': 3,
  'retry_delay': timedelta(minutes=20)
}

dag = DAG('update_dataflows', default_args=default_args, schedule_interval='30 12 * * 3')
```


## 02. Airflow: instanciar un DAG con SLA

SLA para una tarea en especifico
```python
from datetime import timedelta

task1 = BashOperator(task_id='sla_task',
bash_command='runcode.sh',
sla=timedelta(seconds=30),
dag=dag)
```

SLA para el DAG
```python
from datetime import timedelta

default_args={
'sla': timedelta(minutes=20)
'start_date': datetime(2020,2,20)
}
dag = DAG('sla_dag', default_args=default_args)
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

## 03. Airflow: PythonOperator examples
```python
from airflow.operators.python_operator import PythonOperator

def sleep(length_of_time):
  time.sleep(length_of_time)
  
sleep_task = PythonOperator(
task_id='sleep',
python_callable=sleep,
op_kwargs={'length_of_time': 5}
dag=example_dag
)

def pull_file(URL, savepath):
    r = requests.get(URL)
    with open(savepath, 'wb') as f:
        f.write(r.content)   
    # Use the print method for logging
    print(f"File pulled from {URL} and saved to {savepath}")

from airflow.operators.python_operator import PythonOperator

# Create the task
pull_file_task = PythonOperator(
    task_id='pull_file',
    # Add the callable
    python_callable=pull_file,
    # Define the arguments
    op_kwargs={'URL':'http://dataserver/sales.json', 'savepath':'latestsales.json'},
    dag=process_sales_dag
    )

# Add another Python task
parse_file_task = PythonOperator(
    task_id='parse_file',
    # Set the function to call
    python_callable=parse_file,
    # Add the arguments
    op_kwargs={'inputfile':'latestsales.json', 'outputfile':'parsedfile.json'},
    # Add the DAG
    dag=process_sales_dag
)



def printme():
  print("This goes in the logs!")

python_task = PythonOperator(
task_id='simple_print',
python_callable=printme,
dag=example_dag
)
```


## 03. Airflow: EmailOperator example
```python
# Import the Operator
from airflow.operators.email_operator import EmailOperator

# Define the task
email_manager_task = EmailOperator(
    task_id='email_manager',
    to='manager@datacamp.com',
    subject='Latest sales JSON',
    html_content='Attached is the latest sales JSON file as requested.',
    files='parsedfile.json',
    dag=process_sales_dag
)

# Set the order of tasks
pull_file_task >> parse_file_task >> email_manager_task
```

## 03. Airflow: Ejemplo datacamp Cronetab e EmailOperator
```python
import requests
import json
from datetime import datetime
from airflow.models import DAG
from airflow.operators.python_operator import PythonOperator
from airflow.operators.email_operator import EmailOperator


default_args = {
    'owner':'sales_eng',
    'start_date': datetime(2020, 2, 15),
}

process_sales_dag = DAG(dag_id='process_sales', default_args=default_args, schedule_interval='@monthly')


def pull_file(URL, savepath):
    r = requests.get(URL)
    with open(savepath, 'w') as f:
        f.write(r.content)
    print(f"File pulled from {URL} and saved to {savepath}")
    

pull_file_task = PythonOperator(
    task_id='pull_file',
    # Add the callable
    python_callable=pull_file,
    # Define the arguments
    op_kwargs={'URL':'http://dataserver/sales.json', 'savepath':'latestsales.json'},
    dag=process_sales_dag
)

def parse_file(inputfile, outputfile):
    with open(inputfile) as infile:
      data=json.load(infile)
      with open(outputfile, 'w') as outfile:
        json.dump(data, outfile)
        
parse_file_task = PythonOperator(
    task_id='parse_file',
    # Set the function to call
    python_callable=parse_file,
    # Add the arguments
    op_kwargs={'inputfile':'latestsales.json', 'outputfile':'parsedfile.json'},
    # Add the DAG
    dag=process_sales_dag
)

email_manager_task = EmailOperator(
    task_id='email_manager',
    to='manager@datacamp.com',
    subject='Latest sales JSON',
    html_content='Attached is the latest sales JSON file as requested.',
    files='parsedfile.json',
    dag=process_sales_dag
)

pull_file_task >> parse_file_task >> email_manager_task
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


## 06. Airflow: Sensors

Para saber que tipo de `executor`tenemos, podemos hacerlo ejecutando `airflow list_dags` y visualizando su salida:
```bash
bash ejemplo.sh
```

```python
from airflow.models import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.operators.python_operator import PythonOperator
from airflow.operators.http_operator import SimpleHttpOperator
from airflow.contrib.sensors.file_sensor import FileSensor

dag = DAG(
   dag_id = 'update_state',
   default_args={"start_date": "2019-10-01"}
)

precheck = FileSensor(
   task_id='check_for_datafile',
   filepath='salesdata_ready.csv',
   dag=dag)

part1 = BashOperator(
   task_id='generate_random_number',
   bash_command='echo $RANDOM',
   dag=dag
)

import sys
def python_version():
   return sys.version

part2 = PythonOperator(
   task_id='get_python_version',
   python_callable=python_version,
   dag=dag)
   
part3 = SimpleHttpOperator(
   task_id='query_server_for_external_ip',
   endpoint='https://api.ipify.org',
   method='GET',
   dag=dag)
   
precheck >> part3 >> part2
```




# Chapter 02. Implementing Airflow DAGs.

# 5 семинар IT школы

Алгоритм Гровера

## Шаблон

```python
import numpy as np
from qiskit import IBMQ, QuantumCircuit, Aer, execute
from qiskit.quantum_info import Operator
from qiskit.providers.ibmq import least_busy
from qiskit.visualization import plot_histogram
from qiskit.tools.jupyter import *
provider = IBMQ.load_account()
```

Реализация Оракла:

```python
def oracle(n, indices_to_mark, name = 'Oracle'):
```

Реализация diffuser

```python
def diffuser(n):
```

Сам алгоритм Гровера:

```python
def Grover(n, marked):
```

Нарисуем квантовую схему:

```python
n = 5
x = np.random.randint(2**n)
marked = [x]
qc = Grover(n, marked)

qc.draw()
```

Запустим на симуляторе:

```python
backend = Aer.get_backend('qasm_simulator')
result = execute(qc, backend, shots=10000).result()
counts = result.get_counts(qc)
print(counts)
print(np.pi/(4*np.arcsin(np.sqrt(len(marked)/2**n)))-1/2)
plot_histogram(counts)
```

Запустим на квантовой машине:

```python
n = 3
x = np.random.randint(2**n)
y = np.random.randint(2**n)
while y==x:
    y = np.random.randint(2**n)
marked = [x,y]
qc = Grover(n, marked)

backend = least_busy(provider.backends(filters=lambda x: x.configuration().n_qubits == 5 and
                                      not x.configuration().simulator and x.status().operational==True))
print("least busy backend: ", backend)
# backend = provider.get_backend('ibmq_lima')
%qiskit_job_watcher

shots = 1024
job = execute(qc, backend=backend, shots=shots, optimization_level=3)
```

Дождёмся результатов запуска:

```python
results = job.result()
answer = results.get_counts()
plot_histogram(answer)
```

Для просмотра ранее запущенных работ:

```python
backend = provider.get_backend('ibmq_lima')
old_job = backend.retrieve_job('') # job id
results = old_job.result()
answer = results.get_counts()
plot_histogram(answer)
```

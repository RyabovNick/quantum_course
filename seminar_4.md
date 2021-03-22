# 4 семинар IT школы

Алгоритм Дойча-Йожи

## Шаблон

TODO: дополнится кодом, после реализации на семинаре

```python
import numpy as np

from qiskit import IBMQ, BasicAer
from qiskit.providers.ibmq import least_busy
from qiskit import QuantumCircuit, execute
from qiskit.tools.jupyter import *
provider = IBMQ.load_account()

from qiskit.visualization import plot_histogram
```

Реализуем Оракул (Это наш собственный квантовый гейт):

```python
# Создание гейта Oracle
# case - constant || balanced, означает какая функция
# n - количество кубитов
def oracle(case, n):
```

Реализуем сам алгоритм (должен вернуть квантовую схему):

```python
# Deutsch–Jozsa алгоритм
def dj_algorithm(n, case):

```

```python
n = 4
circuit = dj_algorithm(n, 'constant')
circuit.draw()
```

Запуск на встроенном симуляторе:

```python
backend = BasicAer.get_backend('qasm_simulator')
shots = 1024
dj_circuit = dj_algorithm(n, 'constant')
results = execute(dj_circuit, backend=backend, shots=shots).result()
answer = results.get_counts()

plot_histogram(answer)
```

Запуск на квантовой машине:

```python
backend = least_busy(provider.backends(filters=lambda x: x.configuration().n_qubits >= (n+1) and
                                      not x.configuration().simulator and x.status().operational==True))
print("least busy backend: ", backend)
%qiskit_job_watcher
dj_circuit = dj_algorithm(n, 'constant')
job = execute(dj_circuit, backend=backend, shots=shots, optimization_level=3)
```

Просмотр результата:

```python
results = job.result()
answer = results.get_counts()
plot_histogram(answer)
```

Также можно в дальнейшем просмотреть уже выполненные работы:

```python
# Берём нужный бэкенд
backend = provider.get_backend('ibmq_belem')
# и получаем по ID старую job (указать ваш ID)
old_job = backend.retrieve_job('6058401308cfba57bf86211c')
results = old_job.result()
answer = results.get_counts()
plot_histogram(answer)
```

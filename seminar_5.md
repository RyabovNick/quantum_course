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
def phase_oracle(n, indices_to_mark, name = 'Oracle'):
    # создаём квантовую схему Oracle
    qc = QuantumCircuit(n, name=name)
    # создаём матрицу с размерностью кол-ва элементов в входном параметре
    # в итоге нужный элемент должен быть помечен -1, а у всех остальных +1
    # np.identity создаёт сразу матрицу с 1 по диагонали
    oracle_matrix = np.identity(2**n)
    for index_to_mark in indices_to_mark:
        oracle_matrix[index_to_mark, index_to_mark] = -1
    # Преобразуем матрицу в оператор из N кубитов
    # https://qiskit.org/documentation/stubs/qiskit.quantum_info.Operator.html
    # и делаем свой собственный унитарный гейт, т.е.
    qc.unitary(Operator(oracle_matrix), range(n))
    return qc
```

Реализация diffuser

```python
def diffuser(n):
    # Создаём новую квантовую схему
    qc = QuantumCircuit(n, name='Diffuser')
    # Применяем гейт Адамара ко всем кубитам
    qc.h(range(n))
    # Затем применяем написанный ранее Оракл
    # который принимает помеченный элемент
    # 0 - элемент 0 для всех N кубитов
    qc.append(phase_oracle(n,[0]),range(n))
    # и опять применяем гейт Адамара ко всем кубитам
    qc.h(range(n))
    return qc
```

Сам алгоритм Гровера:

```python
def Grover(n, marked):
    # Создаём квантовую схему с N кубитов и N классических битов
    # для итогового измерения кубитов
    qc = QuantumCircuit(n, n)
    # точность или сколько раз мы будем применять Оракл и Диффузор
    # для ситуации, когда помечено несколько элементов нам надо расчитать
    # сколько оптимально будет запустить Ораул и Диффузор
    r = int(np.round(np.pi/(4*np.arcsin(np.sqrt(len(marked)/2**n)))-1/2))
    print(f'{n} qubits, basis state {marked} marked, {r} rounds')
    # Применяем гейт Адамара
    qc.h(range(n))
    for _ in range(r):
        # Применяем Оракл для всех кубитов
        qc.append(phase_oracle(n, marked), range(n))
        # Применяем диффузор, также для всех кубитов
        qc.append(diffuser(n), range(n))
    # и в конце делаем измерение
    qc.measure(range(n), range(n))
    return qc
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
# 10000 попыток сделано для того, чтобы показать, что вероятность не 1
# но это не потому, что тут есть шумы, мы используем симулятор, тут нет никаких шумов
# почему мы не получили вероятность 1 - дело в r - оно 3.91..., а не 4
result = execute(qc, backend, shots=10000).result()
counts = result.get_counts(qc)
print(counts)
print(np.pi/(4*np.arcsin(np.sqrt(len(marked)/2**n)))-1/2)
plot_histogram(counts)
# в результате видим бинарное искомое число
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

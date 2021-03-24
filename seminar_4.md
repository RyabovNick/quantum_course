# 4 семинар IT школы

Алгоритм Дойча-Йожи

## Реализация

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
# case - constant || balanced, означает какую функцию мы ищем
# n - количество кубитов
def oracle(case, n):
    # Создание квантовой схемы с n+1 кубитов
    # N кубит - это вход в оракл, +1 кубит - это кубит Y (выходной, output qubit)
    oracle_qc = QuantumCircuit(n+1)

    # случай, когда функция является сбалансированной
    # на самом деле есть много вариантов поведения, т.е.
    # по сути если на выходе мы не получаем все 0 - это сбалансированная фунция
    # в данном случае делаем, чтобы возвращались все 1
    if case == "balanced":
        # Для каждого кубита, кроме последнего
        for qubit in range(n):
            # Применяем CNOT гейт на последний кубит
            # CNOT переворачивает второй кубит только тогда, когда
            # первый кубит равен 1.
            # В итоге на последний (выходной) кубит будет применяться столько раз
            # CNOT, сколько кубитов (он используется как target)
            # последний кубит - это n, потому что отсчёт с 0
            # все входные кубиты используются как управляющие (control).
            # В итоге в зависимости от того, сколько входных кубитов равны 1
            # столько раз будет измененён выходной кубит.
            # Это только один из вариантов сделать balanced
            oracle_qc.cx(qubit, n)

    # случай, когда функция постоянная, т.е. принимает
    # либо 0, либо 1 при любых аргументах
    #
    # По сути если мы хотим получить все 0, то нам не надо делать ничего
    # Т.е. Два гейта Адамара как бы обнуляют действие друг друга и возвращают
    # кубиты в значение 0.
    if case == "constant":
        # случайным образом определяем чему будет равна функция (0 или 1)
        output = np.random.randint(2)
        if output == 1:
            # Паули гейт: меняет кубит на противоположное значение
            oracle_qc.x(n)

    # to_gate создаёт гейт на основе схемы
    oracle_gate = oracle_qc.to_gate()
    oracle_gate.name = "Oracle"
    return oracle_gate
```

Реализуем сам алгоритм (должен вернуть квантовую схему):

```python
# Deutsch–Jozsa алгоритм
def dj_algorithm(n, case):
    # Создание схемы с n+1 кубитами и n - классическими битами
    # Классические биты нужны для того, чтобы хранить результат
    # измерения кубитов. Во время измерения необходимо положить
    # результат в классический бит.
    # Последний кубит не измеряется, потому что он нужен
    # только для построение Oracle.
    circuit = QuantumCircuit(n+1, n)

    # На каждый входной кубит применяется гейт Адамара
    for qubit in range(n):
        circuit.h(qubit)

    # на последний кубит применяется Паули гейт и гейт Адамара
    circuit.x(n)
    circuit.h(n)

    # вызывается написанная ранее функция oracle
    dj_oracle = oracle(case, n)
    # И на все кубиты применяется наш Oracle гейт
    circuit.append(dj_oracle, range(n+1))

    # для каждого кубита, кроме последнего
    for i in range(n):
        # применяем гейт Адамара
        circuit.h(i)
        # и измеряем кубит
        circuit.measure(i, i)

    return circuit
```

```python
n = 4
circuit = dj_algorithm(n, 'constant')
circuit.draw()
```

Запуск на встроенном симуляторе:

```python
backend = BasicAer.get_backend('qasm_simulator')
# для запуске на симуляторе это не сильно важно, потому что шумов нет
# а вот для дальнейшего запуска на настоящем устройстве это необходимо,
# чтобы видеть влияние шумов
shots = 1024
dj_circuit = dj_algorithm(n, 'balanced')
results = execute(dj_circuit, backend=backend, shots=shots).result()
answer = results.get_counts()

plot_histogram(answer)
```

Запуск на квантовой машине:

```python
backend = least_busy(provider.backends(filters=lambda x: x.configuration().n_qubits >= (n+1) and
                                      not x.configuration().simulator and x.status().operational==True))
print("least busy backedn: ", backend)
%qiskit_job_watcher
dj_circuit = dj_algorithm(n, 'balanced')
# optimization_level - т.к. много CNOT гейта, будет использовать максимальный
# уровень оптимизации, для связанных кубитов
# Если мы делаем эту операция на несвязанные кубиты, то вероятность ошибок
# возрастает, т.е. приходится делать много swap (изменений), чтоб сделать
# операцию на эти 2 кубита. И уровень оптимизации немного сглаживает эту проблему
job = execute(dj_circuit, backend=backend, shots=shots, optimization_level=3)
```

Просмотр результата:

```python
# Если мы запустим, мы увидим совершенно другую картину по сравнению с симулятором
# т.к. имеет много шумов. Вероятность не такая большая, но мы можем
# идентифицировать ответ. Мы сделали 1024 попытки, чтобы получить
# максимально правильный ответ, потому что если мы сделаем 1 shot

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

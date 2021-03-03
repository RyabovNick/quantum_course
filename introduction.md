# Qiskit Introduction

```python
from qiskit import QuantumCircuit, assemble, Aer
from qiskit.visualization import plot_histogram

n = 8
n_q = n
n_b = n
qc_output = QuantumCircuit(n_q,n_b)

for j in range(n):
    qc_output.measure(j,j)
```

Для визуализации квантовой схемы:

```python
qc_output.draw()
```

Кубиты всегда инициализируются 0, т.к. над ними не производится никаких операций, то результат будет 0.

```python
# симулятор из Qiskit
sim = Aer.get_backend('qasm_simulator')
# https://qiskit.org/documentation/stubs/qiskit.compiler.assemble.html сериализирует payload, принимает на вход квантовую схему, возвращает Qobj, который может быть запущен на любом бэкенде (симулятор или системе)
qobj = assemble(qc_output)
counts = sim.run(qobj).result().get_counts()
plot_histogram(counts)
```

```python
qc_encode = QuantumCircuit(n)
qc_encode.x(7)
qc_encode.draw()
```

Квантовые схемы можно соединять:

```python
qc = qc_encode + qc_output
qc.draw()
```

```python
qc_encode = QuantumCircuit(n)
qc_encode.x(0)
qc_encode.x(1)
qc_encode.x(4)

qc_encode.draw()
```

Для многих задач также полезен XOR gate

|  A  |  B  | XOR |
| :-: | :-: | :-: |
|  0  |  0  |  0  |
|  0  |  1  |  1  |
|  1  |  0  |  1  |
|  1  |  1  |  0  |

```python
qc_cnot = QuantumCircuit(2)
qc_cnot.cx(0,1)
qc_cnot.draw()
```

```python
qc = QuantumCircuit(2,2)
qc.x(0)
qc.cx(0,1)
qc.measure(0,0)
qc.measure(1,1)
qc.draw()
```

```python
# результат вычисления мы хотим положить в другие кубиты, поэтому 2 кубита - для входа, 2 будем использовать в вычислениях, итого 4. И для результата понадобятся только 2 классических бита
qc_ha = QuantumCircuit(4,2)
# кодируем входящие кубиты
qc_ha.x(0) # Закомментируйте, чтобы A = 0
qc_ha.x(1) # Закомментируйте, чтобы B = 0
qc_ha.barrier() # для визуального более удобного разделения этапов на схеме
# CNOTs для кубита 2. Кубит 2 - это то, что будет записано в конце (т.е. второй классический бит). Мы можем использовать 2 CNOT гейта на 0 и 2, а затем на 1 и 2 кубиты, что вычислить, чему будет равен кубит 2
qc_ha.cx(0,2)
qc_ha.cx(1,2)
qc_ha.barrier()
# вычисляем
qc_ha.measure(2,0)
qc_ha.measure(3,1)

qc_ha.draw()
```

```python
# результат вычисления мы хотим положить в другие кубиты, поэтому 2 кубита - для входа, 2 будем использовать в вычислениях, итого 4. И для результата понадобятся только 2 классических бита
qc_ha = QuantumCircuit(4,2)
# кодируем входящие кубиты
qc_ha.x(0) # Закомментируйте, чтобы A = 0
qc_ha.x(1) # Закомментируйте, чтобы B = 0
qc_ha.barrier() # для визуального более удобного разделения этапов на схеме
# CNOTs для кубита 2. Кубит 2 - это то, что будет записано в конце (т.е. второй классический бит). Мы можем использовать 2 CNOT гейта на 0 и 2, а затем на 1 и 2 кубиты, что вычислить, чему будет равен кубит 2
qc_ha.cx(0,2)
qc_ha.cx(1,2)
# Toffoli, чтобы вычислить 3-ий кубит (1-ый классический бит)
qc_ha.ccx(0,1,3)
qc_ha.barrier()
# вычисляем
qc_ha.measure(2,0)
qc_ha.measure(3,1)

qc_ha.draw()
```

## Кубиты

```python
from qiskit import QuantumCircuit, assemble, Aer
from qiskit.visualization import plot_histogram, plot_bloch_vector
from math import sqrt, pi

qc = QuantumCircuit(1) # Создание квантовой схемы с одним кубитом
initial_state = [0,1]   # Определяем начальное состояние как |1>
qc.initialize(initial_state, 0) # Применяем к 0 кубиту
qc.draw()  # Рисуем схему

svsim = Aer.get_backend('statevector_simulator') # Берём симулятор

qobj = assemble(qc)     # Создаём Qobj
result = svsim.run(qobj).result() # Симулируем и возвращаем результат симуляции

out_state = result.get_statevector()
print(out_state) # Отобразим вектор состояния
```

Python j - это i для обозначения комплексных чисел. Т.е. тут это [0.+0.j 1.+0.j], где 0.+0.j - 0, 1.+0.j - 1

Измерим кубиты:

```python
qc.measure_all()
qc.draw()
```

Получим вероятность, а не вектор состояния и нарисуем:

```python
counts = result.get_counts()
plot_histogram(counts)
```

# Geolocation

## Формальная постановка задачи

Разработать алгоритм определения геолокации объекта: попадание точки в один из **пересекающихся** интервалов. Необходимо вернуть интервал с наибольшим весом. 

#### Пояснения к условию

* Данные интервалы - это отрезки на оси Х, граничные точки которых включаются в интервал. 
* Проверяемая точка - это число, которое попадает или не попадает в один из интервалов.
* Веса интервалов задаются для каждого из них во входном файле `input.txt`, формат которого будет описан [тут](README.md)
* Результат программы записывается в выходном файле `result.txt`

## Общий план решения

1. Считать входные данные из файла `input.txt`.
2. Сохранить их в некоторую структуру данных (размышления о выборе которой приводятся ниже).
3. Каким-то особым образом, зависящим от выбранной структуры, пройтись по ней, чтобы найти все подходящие интервалы, куда попадает контрольная точка.
4. Выбрать из них интервал максимального веса.
5. Записать результат в выходной файл `result.txt`.

## Выбор структуры данных

В этой части ***интервалом*** будет называться объект класса `Interval`, содержащего поля координат: `begin_point` и `end_point`, а так же поле веса интервала: `weight`.

Также будем считать, что на входе нам задано `n` различных интервалов.

Рассмотрим несколько вариантов выбора структуры данных для хранения интервалов.

* ### Массив
    
Для размещения n элементов в массив понадобится **O(*n*)** времени. (Выделение памяти - O(*1*), вставка элемента *n* раз - *n* * O(*1*) = O(*n*))
    
Для более удобного поиска результирующего интервала массив нужно отсортировать по весу каждого из интервалов. (оценка времени сортировки - **O(*n log n*)**)
    
После этого почти сразу можно найти нужный нам интервал. Это можно сделать, один раз пробежавшись по всем элементам массива (за время **O(*n*)**), с каждым элементом проводя следующие операции:
    
1) Если интервал не подходит заданной точке, берем следующий.
2) Если интервал подходит, записать его в некоторую переменную `result_interval` и перейти к следующему.

#### Вывод:
    
Чтобы найти необходимый интервал, нужно перебрать все имеющиеся интервалы. Но если ***n*** - большое число, а искать интервалы, соответствующие разным точкам, приходится часто, перебор всех значений (***brute-force***) окажется совсем не оптимальным решением.

* ### Двоичное Дерево Поиска (Binary Search Tree)
	
Если бы мы рассматривали простой случай данной задачи, когда интервалы **не пересекаются**, они могли бы храниться в ДДП, а поиск нужного интервала в среднем производился бы за **O(*log n*)** (но поиск в худшем все равно занимал бы **O(*n*)** времени). 
    
Но в условии нашей задачи сказано, что интервалы **могут пересекаться**, из-за чего нельзя будет реализовать однозначно-корректную вставку элемента в ДДП.
    
Например, потому что нельзя сравнить два интервала, у которых порядок между начальными координатами отличается от порядка между конецными координатами. *(Как сравнить интервалы (3; 7) и (4; 6)?)*
    
Чтобы избежать данного конфликта можно построить два дерева: одно - отсортированное по начальной координате, второе - по конечной. 
    
Такое построение позволит при поиске подходящего интервала обойти каждое дерево в среднем за **O(*log n*)** времени, но результаты необходимо соединить между собой, что потребует **O(*n*)** времени.
    
#### Вывод:

Получается, что использование ДДП для поиска подходящего интервала ничем не оптимальнее полного перебора.
    
Как мы видим, нужно придумать что-то получше ДДП.
    
### Интервальное дерево (Centered Interval Tree)
	
**Интервальное дерево** - деревовидная структура данных, предназначенная для хранения интервалов. Его особенностью является то, что оно позволяет эффективно искать все интервалы, перекрывающие заданную точку или же другой интервал.
    
#### Построение дерева
    
1) Берется диапазон всех интервалов и делится пополам, таким образом получая координату `x_center`, которая подбирается так, чтобы дерево было относительно сбалансированно.
    
    Другой известный способ подобрать координату `x_center` - принять ее равной медиане отсортированной последовательности всех концов интервалов.

2) Выделяется три группы интервалов:

	* `left_int` - те, что находятся строго левее `x_center`
	* `right_int` - те, что находятся строго правее `x_center`
	* `center_int` - те, которые содержат `x_center`
	
3) Интервалы, попавшие в `center_int`, сохраняются о специальную структуру данных, состоящей из двух списков, один из которых хранит эти интервалы, отсортированные по левой координате, а второй - по правой.

4) Для `left_int` и `right_int` рекурсивно строится дерево по пунктам 1., 2. и 3.

#### Оценки времени построения и памяти
    
* Высота интервального дерева имеет асимптотику **O(*log n*)**, т.к. при каждом рекурсивном вызове построения дерева множество концов отрезков уменьшается более чем в 2 раза, следовательно после O(*log n*) рекурсивных вызовов множетсво интервалов будет пусто.

* Вершин в интервальном дереве - **O(*n*)**

* Дерево интервалов занимает **O(*n*)** памяти (необходимо хранить O(*n*) вершин и каждый из *n* интервалов в двух списках).

* Дерево интервалов строится за **O(*n log n*)** времени.

    Суммарное время для создания списков интервалов - O(*n log n*), т.к. их суммарная длина на каждом уровне высоты дерева асимптотически равна O(*n*). 
        
    Необходимо добавить время, которое затратится на все рекурсивные вызовы `set_intervals`, уже без учета сортировок. 
    
    Для каждой вершины это время равно O(*m*), где *m* - количество интервалов, которые были переданы в нее рекурсивно. m <= n 
    
    Если просуммировать это время по всем O(*log n*) слоям дерева (в каждом слое лежит O(*n*) отрезков), получится O(*n log n*).
        
#### Ответ на запрос с координатой точки
    
1) Поиск списка подходящих точке `p` отрезков начинается с корня дерева.
2) Если `p == x_center` текущего узла, то возвращаем один из списков, хранящихся в `center_int`.
3) Если `p > x_center`, тогда в результирующую переменную `result_intervals` нужно добавить какое-то количество интервалов из отсортированного по правой координате списка из `center_int` и перейти к поиску в правом поддереве. 
4) Если `p < x_center`, тогда в результирующую переменную `result_intervals` нужно добавить какое-то количество интервалов из отсортированного по левой координате списка из `center_int` и перейти к поиску в левом поддереве.

#### Оценка времени ответа на запрос с координатой точки
    
Ответ на запрос происходит за **O(*log n + answer*)** времени, где *answer* - колчество подходящих интервалов
    
*-* Почему?    
    
*-* Т.к. высота дерева интервалов асимптотически равна O(*log n*), значит рекурсивных вызовов происходит тоже O(*log n*). В каждой вершине добавление интервалов в результирующий массив происходит за O(answer), т.к. в каждой вершине может быть просмотрен только 1 лишний интервал.
    
#### Вывод
    
В данной конкретной задаче я остановила свой выбор именно на интервальном дереве, т.к. в среднем оно будет давать выигрыш по времени по сравнению со временем, необходимым на полным перевобор.

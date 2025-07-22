# 람다 표현식 


### 짝수만 걸러내기 
```python
nums = [1, 2, 3, 4, 5, 6]

even_nums = list(filter(lambda x: x % 2 == 0, nums))
print(even_nums)  # [2, 4, 6]
```

# 빈 문자열 제거
```python
words = ["hello", "", "world", "", "python"]

non_empty = list(filter(lambda x: x != "", words))
print(non_empty)  # ['hello', 'world', 'python']
```

# None, False 제거 (falsy 값 제거)
```python
data = [0, 1, "", "text", [], [1, 2], None, True, False]

clean = list(filter(None, data))
print(clean)  # [1, 'text', [1, 2], True]
```

# map 람다
```python
arr_1 = list(map(lambda x: x, data_1))
print(arr_1)

```
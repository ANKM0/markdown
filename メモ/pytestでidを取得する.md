# pytest idを取得する

test001とtest002で期待値を変えたい場合とか

```py
def calculate_avg(a: int, b: int) -> int:
  result  = a + b / 2
```

## 参考

- <https://stackoverflow.com/questions/56466111/how-to-find-the-test-id-for-parametrized-tests-during-execution-of-the-test>
- <https://docs.pytest.org/en/stable/reference/reference.html#request>
- <https://hackebrot.github.io/pytest-tricks/param_id_func/>

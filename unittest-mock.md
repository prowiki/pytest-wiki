# ToC: unittest.mock

- [ToC: unittest.mock](#toc-unittestmock)
  - [Preface](#preface)
  - [unittest.mock.Mock](#unittestmockmock)
  - [unittest.mock.patch](#unittestmockpatch)
    - [patch object](#patch-object)
    - [patch function directly](#patch-function-directly)
    - [patch a long chain of function call](#patch-a-long-chain-of-function-call)
    - [side_effect](#side_effect)
    - [patch dict](#patch-dict)

## Preface

1. Don't use `MagicMock` unless you know what you are doing: https://stackoverflow.com/questions/17181687/mock-vs-magicmock. But in most time, there's no difference as long as you don't test "magic" methods.
2. Sometimes you might use `MagicMock` unintentionally, e.x. when you are using `patch.object()`.
3. Ref: https://mozillazg.com/2018/03/python-introduction-mock-unit-test-examples-cookbook.html. There are some usages, such as mock properties and builtin functions that we don't cover in this wiki.

## unittest.mock.Mock

```python
from unittest.mock import Mock, patch

m = Mock()
m.func.return_value = 'hello'
print(m.func())
print(m.func(2))
```

## unittest.mock.patch

### patch object

```python
# basic usage: return value
with patch.object(time, 'time', return_value='now~'):
    print(time.time())

# but I prefer this way
with patch.object(time, 'time') as mock_time:
    mock_time.return_value = 'now~'
    print(time.time(1))
    print(time.time())
    print("called?:", mock_time.called)
    print("call_count:", mock_time.call_count)
```

### patch function directly

```python
# patch function
with patch('time.time', Mock(return_value='now~')):
    print(time.time())
```

### patch a long chain of function call

```python
# my preferable way to mock a long chain of function call
with patch.object(time, 'time') as mock_time:
    print(type(mock_time))
    # be attention to the first 'return_value'
    mock_time.return_value.my_func(...).get_value.return_value = 'now~now~'
    # mock_time.return_value = new_mock = Mock()
    # new_mock.my_func(...).get_value.return_value = 'now~now~'
    print(time.time().my_func(1).get_value())
```

If a function is used in many tests, we can combine it with `pytest.fixture`.

```python
@pytest.fixture
def mock_time_fixture():
    print("before fixture")
    with patch.object(time, 'time') as mock_time:
        # can't use return, because yield keep the context
        yield mock_time.return_value
    print("after fxiture")

def test_mock(mock_time_fixture):
    print(">>>>>", type(mock_time_fixture))
    mock_time_fixture.my_func(...).get_value.return_value = 'now~now~'
    print(time.time().my_func(1).get_value())
```

### side_effect

If you want the function return different across different calls, then `side_effect` is what you need!

```python
with patch.object(time, 'time', side_effect=['now~', 'a second later~']):
    print(time.time())
    print(time.time())

with patch.object(time, 'time', side_effect=range(10)):
    print(time.time())
    print(time.time())
```

Also, it can mock a exception.

```python
with patch.object(time, 'time', side_effect=ValueError('side effect error')):
    print(time.time())
```

### patch dict

```python
my_dict = {'key': 'value'}
with patch.dict(my_dict, {'key': 'new_val'}):
    print(my_dict['key'])

with patch.dict(my_dict, key='new_val', ext='ext_val'):
    print(my_dict['key'])
    print(my_dict['ext'])
```

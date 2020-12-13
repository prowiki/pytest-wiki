# ToC: pytest

- [ToC: pytest](#toc-pytest)
  - [pytest basics](#pytest-basics)
    - [Reference](#reference)
    - [Installation](#installation)
    - [Usage](#usage)
    - [Catching Exceptions](#catching-exceptions)
    - [Mark](#mark)
      - [Custom Marks](#custom-marks)
      - [Built-in Marks: skip](#built-in-marks-skip)
      - [Built-in Marks: xfail](#built-in-marks-xfail)
      - [Parametrize](#parametrize)
    - [Fixture](#fixture)
      - [Before test cases](#before-test-cases)
      - [After test cases](#after-test-cases)
      - [Auto use fixtures](#auto-use-fixtures)
      - [Parametrize fixture](#parametrize-fixture)
      - [Built-in fixtures](#built-in-fixtures)

## pytest basics

`pytest` is a 3rd-party lib built based on `unittest`.

### Reference

1. https://learning-pytest.readthedocs.io/zh/latest/index.html
2. https://docs.pytest.org/en/latest/monkeypatch.html

### Installation

```bash
pip install -U pytest
```

### Usage

1. run all tests in a file: `pytest <test_file_path>`.
2. with a specific test case: `-k <test_case_name>` or `<test_file_path>::<test_case_name>`.
3. show `print()`: `-s`.
4. only run *marked* test cases: `-m "<marks>"`.
5. show verbose: `-v`.

### Catching Exceptions

```python
# test_raises.py

import pytest

def foo():
    raise ValueError("a value error")

def test_raises():
    with pytest.raises(ValueError) as e:
        foo()
    exec_msg = e.value.args[0]
    assert exec_msg == 'a value error'
```

### Mark

#### Custom Marks

```python
# test_with_mark.py

import pytest

# custom mark name: @pytest.mark.<name>
@pytest.mark.finished
def test_func1():
    assert 1 == 1

@pytest.mark.unfinished
def test_func2():
    assert 1 != 1
```

1. Run tests with given tasks: `pytest test_with_mark.py -m finished`.
2. mark with logic calculation: `-m "finished or unfinished"`, `-m "finished and commit"`

#### Built-in Marks: skip

1. automatically skip tests: `@pytest.mark.skip(reason='out-of-date api')`;
2. skip tests with condition: `@pytest.mark.skipif(conn.__version__ < '0.2.0')`

#### Built-in Marks: xfail

We expect the test to fail.
If it fails, results will show `x`(xfail); If it succeeds, results will show `X`(xpass).

```python
# test_xfail.py

import pytest

version = '0.1.0'

@pytest.mark.xfail(version < '0.2.0',
                   reason='not supported until v0.2.0')
def test_api():
    id_1 = 1
    id_2 = 1
    assert id_1 != id_2
```

#### Parametrize

```python
# test_parametrize.py

import pytest

@pytest.mark.parametrize('user, passwd',
                         [('jack', 'abcdefgh'),
                          ('tom', 'a123456a')])
def test_passwd_md5(user, passwd):
    db = {
        'jack': 'e8dc4081b13434b45189a720b77b6818',
        'tom': '1702a132e769a623c1adb78353fc9503'
    }

    import hashlib

    assert hashlib.md5(passwd.encode()).hexdigest() == db[user]
```

It will run this test case twice with two `passwd`s, respectively.

### Fixture

Fixtures are functions ran before or after test cases.

#### Before test cases

```python
# test_postcode.py

import pytest

@pytest.fixture()
def postcode():
    return '010'

def test_postcode(postcode):
    assert postcode == '010'
```

#### After test cases

```python
# test_db.py

import pytest

@pytest.fixture()
def db():
    print('Connection successful')
    yield
    print('Connection closed')

def search_user(user_id):
    d = {
        '001': 'xiaoming'
    }
    return d[user_id]

def test_search(db):
    assert search_user('001') == 'xiaoming'
```

Note: use `-s` to show printed contents.

#### Auto use fixtures

Typically you just want for setup/teardown only and don't want the return values.

```python
# test_autouse.py

import time
import pytest

@pytest.fixture(autouse=True)
def timer_function_scope():
    start = time.time()
    yield
    print(' Time cost: {:.3f}s'.format(time.time() - start))

def test_1():
    time.sleep(1)

def test_2():
    time.sleep(2)
```

Note: use `-s` to show printed contents.

#### Parametrize fixture

```python
# test_parametrize.py

import pytest

@pytest.fixture(params=[
    ('redis', '6379'),
    ('elasticsearch', '9200')
])
def param(request):
    return request.param

@pytest.fixture(autouse=True)
def db(param):
    print('\nSucceed to connect %s:%s' % param)
    yield
    print('\nSucceed to close %s:%s' % param)

def test_api():
    assert 1 == 1
```

Note: use `-s` to show printed contents.

#### Built-in fixtures

1. `tmpdir`: temporary path

```python
def test_tmpdir(tmpdir):
    a_dir = tmpdir.mkdir('mytmpdir')
    a_file = a_dir.join('tmpfile.txt')

    a_file.write('hello, pytest!')

    assert a_file.read() == 'hello, pytest!'
```

```python
import json

def test_tmpdir(tmpdir):
    a_dir = tmpdir.mkdir('mytmpdir')
    a_file = a_dir.join('tmpfile.txt')

    d = {"a": 1, "b": 2}

    with open(a_file, "w") as fp:
        json.dump(d, fp)

    with open(a_file, "r") as fp:
        ed = json.load(fp)

    assert d == ed
```

Another option is `tmpdir_factory`.

2. `copsys`: read from system out/err

```python
# test_capsys.py

import sys
import pytest

def ping(output):
    print('Pong...', file=output)

def test_stdout(capsys):
    ping(sys.stdout)
    out, err = capsys.readouterr()
    assert out == 'Pong...\n'
    assert err == ''

def test_stderr(capsys):
    ping(sys.stderr)
    out, err = capsys.readouterr()
    assert out == ''
    assert err == 'Pong...\n'
```

3. `monkeypatch`

Functions provided in `monkeypatch`:

```python
setattr(target, name, value, raising=True)
delattr(target, name, raising=True)
setitem(dic, name, value)
delitem(dic, name, raising=True)
setenv(name, value, prepend=None)
delenv(name, raising=True)
```

To use `monkeypath` context is recommended.

```python
import pytest
import time
import os

def test_patch(monkeypatch):
    now = time.time()
    def mock_return():
        return now
    monkeypatch.setattr(time, "time", mock_return)
    assert time.time() == now

def test_context(monkeypatch):
    with monkeypatch.context() as m:
        m.setattr(time, "time", "haoxu")
        assert time.time == "haoxu"
    assert time.time != "haoxu"

def test_env(monkeypatch):
    with monkeypatch.context() as m:
        m.setenv("NAME", "hxu")
        assert os.getenv("NAME") == "hxu"

def test_dict(monkeypatch):
    d = {"a": 10, "b": 20}
    with monkeypatch.context() as m:
        m.setitem(d, "a", 1)
        m.delitem(d, "b")
        assert len(d) == 1
        assert d["a"] == 1
    assert len(d) == 2
```

Another option is `unittest.mock`. Here's a comparison/discussion: https://github.com/pytest-dev/pytest/issues/4576.

Also, I have a wiki for `unittest.mock`: https://github.com/prowiki/pytest-wiki/blob/master/unittest-mock.md

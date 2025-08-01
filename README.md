<h1 align="center">optype</h1>

<p align="center">
    Building blocks for precise & flexible type hints.
</p>

<p align="center">
    <a href="https://pypi.org/project/optype/">
        <img
            alt="optype - PyPI"
            src="https://img.shields.io/pypi/v/optype?style=flat"
        />
    </a>
    <a href="https://anaconda.org/conda-forge/optype">
        <img
            alt="optype - conda-forge"
            src="https://anaconda.org/conda-forge/optype/badges/version.svg"
        />
    </a>
    <a href="https://github.com/jorenham/optype">
        <img
            alt="optype - Python Versions"
            src="https://img.shields.io/pypi/pyversions/optype?style=flat"
        />
    </a>
    <a href="https://scientific-python.org/specs/spec-0000/">
        <img
            alt="optype - SPEC 0 — minimum supported dependencies"
            src="https://img.shields.io/badge/SPEC-0-green?style=flat&labelColor=%23004811&color=%235CA038"
        />
    </a>
    <a href="https://github.com/jorenham/optype">
        <img
            alt="optype - license"
            src="https://img.shields.io/github/license/jorenham/optype?style=flat"
        />
    </a>
</p>
<p align="center">
    <a href="https://github.com/jorenham/optype/actions?query=workflow%3ACI">
        <img
            alt="optype - CI"
            src="https://github.com/jorenham/optype/workflows/CI/badge.svg"
        />
    </a>
    <a href="https://github.com/python/mypy">
        <img
            alt="scipy-stubs - mypy"
            src="https://www.mypy-lang.org/static/mypy_badge.svg"
        />
    </a>
    <a href="https://detachhead.github.io/basedpyright">
        <img
            alt="optype - basedpyright"
            src="https://img.shields.io/badge/basedpyright-checked-42b983"
        />
    </a>
    <a href="https://github.com/astral-sh/ruff">
        <img
            alt="optype - code style: ruff"
            src="https://img.shields.io/endpoint?style=flat&url=https://raw.githubusercontent.com/astral-sh/ruff/main/assets/badge/format.json"
        />
    </a>
</p>

---

## Installation

### PyPI

Optype is available as [`optype`][PYPI] on PyPI:

```shell
pip install optype
```

For optional [NumPy][NUMPY] support, it is recommended to use the
`numpy` extra.
This ensures that the installed `numpy` version is compatible with
`optype`, following [NEP 29][NEP29] and [SPEC 0][SPEC0].

```shell
pip install "optype[numpy]"
```

See the [`optype.numpy` docs](#optypenumpy) for more info.

### Conda

Optype can also be installed with `conda` from the [`conda-forge`][CONDA] channel:

```shell
conda install conda-forge::optype
```

[PYPI]: https://pypi.org/project/optype/
[CONDA]: https://anaconda.org/conda-forge/optype
[NUMPY]: https://github.com/numpy/numpy

## Example

Let's say you're writing a `twice(x)` function, that evaluates `2 * x`.
Implementing it is trivial, but what about the type annotations?

Because `twice(2) == 4`, `twice(3.14) == 6.28` and `twice('I') = 'II'`, it
might seem like a good idea to type it as `twice[T](x: T) -> T: ...`.
However, that wouldn't include cases such as `twice(True) == 2` or
`twice((42, True)) == (42, True, 42, True)`, where the input- and output types
differ.
Moreover, `twice` should accept *any* type with a custom `__rmul__` method
that accepts `2` as argument.

This is where `optype` comes in handy, which has single-method protocols for
*all* the builtin special methods.
For `twice`, we can use `optype.CanRMul[T, R]`, which, as the name suggests,
is a protocol with (only) the `def __rmul__(self, lhs: T) -> R: ...` method.
With this, the `twice` function can written as:

<table>
<tr>
<th width="415px">Python 3.11</th>
<th width="415px">Python 3.12+</th>
</tr>
<tr>
<td>

```python
from typing import Literal
from typing import TypeAlias, TypeVar
from optype import CanRMul

R = TypeVar("R")
Two: TypeAlias = Literal[2]
RMul2: TypeAlias = CanRMul[Two, R]


def twice(x: RMul2[R]) -> R:
    return 2 * x
```

</td>
<td>

```python
from typing import Literal
from optype import CanRMul

type Two = Literal[2]
type RMul2[R] = CanRMul[Two, R]


def twice[R](x: RMul2[R]) -> R:
    return 2 * x
```

</td>
</tr>
</table>

But what about types that implement `__add__` but not `__radd__`?
In this case, we could return `x * 2` as fallback (assuming commutativity).
Because the `optype.Can*` protocols are runtime-checkable, the revised
`twice2` function can be compactly written as:

<table>
<tr>
<th width="415px">Python 3.11</th>
<th width="415px">Python 3.12+</th>
</tr>
<tr>
<td>

```python
from optype import CanMul

Mul2: TypeAlias = CanMul[Two, R]
CMul2: TypeAlias = Mul2[R] | RMul2[R]


def twice2(x: CMul2[R]) -> R:
    if isinstance(x, CanRMul):
        return 2 * x
    else:
        return x * 2
```

</td>
<td>

```python
from optype import CanMul

type Mul2[R] = CanMul[Two, R]
type CMul2[R] = Mul2[R] | RMul2[R]


def twice2[R](x: CMul2[R]) -> R:
    if isinstance(x, CanRMul):
        return 2 * x
    else:
        return x * 2
```

</td>
</tr>
</table>

See [`examples/twice.py`](examples/twice.py) for the full example.

## Reference

The API of `optype` is flat; a single `import optype as opt` is all you need
(except for `optype.numpy`).

<!-- TOC start (generated with https://github.com/derlin/bitdowntoc) -->

- [`optype`](#optype)
  - [`Just`](#just)
  - [Builtin type conversion](#builtin-type-conversion)
  - [Rich relations](#rich-relations)
  - [Binary operations](#binary-operations)
  - [Reflected operations](#reflected-operations)
  - [Inplace operations](#inplace-operations)
  - [Unary operations](#unary-operations)
  - [Rounding](#rounding)
  - [Callables](#callables)
  - [Iteration](#iteration)
  - [Awaitables](#awaitables)
  - [Async Iteration](#async-iteration)
  - [Containers](#containers)
  - [Attributes](#attributes)
  - [Context managers](#context-managers)
  - [Descriptors](#descriptors)
  - [Buffer types](#buffer-types)
- [`optype.copy`](#optypecopy)
- [`optype.dataclasses`](#optypedataclasses)
- [`optype.inspect`](#optypeinspect)
- [`optype.io`](#optypeio)
- [`optype.json`](#optypejson)
- [`optype.pickle`](#optypepickle)<!-- - - [`optype.rick`](#optyperick) -->
- [`optype.string`](#optypestring)
- [`optype.typing`](#optypetyping)
  - [`Any*` type aliases](#any-type-aliases)
  - [`Empty*` type aliases](#empty-type-aliases)
  - [Literal types](#literal-types)
- [`optype.dlpack`](#optypedlpack)
- [`optype.numpy`](#optypenumpy)
  - [Shape-typing](#shape-typing)
  - [Array-likes](#array-likes)
  - [Literals](#literals)
  - [`compat` submodule](#compat-submodule)
  - [`random` submodule](#random-submodule)
  - [`Any*Array` and `Any*DType`](#anyarray-and-anydtype)
  - [Low-level interfaces](#low-level-interfaces)

<!-- TOC end -->

### `optype`

There are five flavors of things that live within `optype`,

- The `optype.Just[T]` and its `optype.Just{Int,Float,Complex}` subtypes only accept
  instances of the type itself, while rejecting instances of strict subtypes.
  This can be used to e.g. work around the `float` and `complex`
  [type promotions][BAD], annotating `object()` sentinels with `Just[object]`,
  rejecting `bool` in functions that accept `int`, etc.
- `optype.Can{}` types describe *what can be done* with it.
  For instance, any `CanAbs[T]` type can be used as argument to the `abs()`
  builtin function with return type `T`. Most `Can{}` implement a single
  special method, whose name directly matches that of the type. `CanAbs`
  implements `__abs__`, `CanAdd` implements `__add__`, etc.
- `optype.Has{}` is the analogue of `Can{}`, but for special *attributes*.
  `HasName` has a `__name__` attribute, `HasDict` has a `__dict__`, etc.
- `optype.Does{}` describe the *type of operators*.
  So `DoesAbs` is the type of the `abs({})` builtin function,
  and `DoesPos` the type of the `+{}` prefix operator.
- `optype.do_{}` are the correctly-typed implementations of `Does{}`. For
  each `do_{}` there is a `Does{}`, and vice-versa.
  So `do_abs: DoesAbs` is the typed alias of `abs({})`,
  and `do_pos: DoesPos` is a typed version of `operator.pos`.
  The `optype.do_` operators are more complete than `operators`,
  have runtime-accessible type annotations, and have names you don't
  need to know by heart.

The reference docs are structured as follows:

All [typing protocols][PC] here live in the root `optype` namespace.
They are [runtime-checkable][RC] so that you can do e.g.
`isinstance('snail', optype.CanAdd)`, in case you want to check whether
`snail` implements `__add__`.

Unlike`collections.abc`, `optype`'s protocols aren't abstract base classes,
i.e. they don't extend `abc.ABC`, only `typing.Protocol`.
This allows the `optype` protocols to be used as building blocks for `.pyi`
type stubs.

[BAD]: https://typing.readthedocs.io/en/latest/spec/special-types.html#special-cases-for-float-and-complex
[PC]: https://typing.readthedocs.io/en/latest/spec/protocol.html
[RC]: https://typing.readthedocs.io/en/latest/spec/protocol.html#runtime-checkable-decorator-and-narrowing-types-by-isinstance

#### `Just`

`Just` is an invariant type "wrapper", where `Just[T]` only accepts instances of `T`,
and rejects instances of any strict subtypes of `T`.

Note that e.g. `Literal[""]` and `LiteralString` are not a strict `str` subtypes,
and are therefore assignable to `Just[str]`, but instances of `class S(str): ...`
are **not** assignable to `Just[str]`.

Disallow passing `bool` as `int`:

```py
import optype as op


def assert_int(x: op.Just[int]) -> int:
    assert type(x) is int
    return x


assert_int(42)  # ok
assert_int(False)  # rejected
```

Annotating a sentinel:

```py
import optype as op

_DEFAULT = object()


def intmap(
    value: int,
    # same as `dict[int, int] | op.Just[object]`
    mapping: dict[int, int] | op.JustObject = _DEFAULT,
    /,
) -> int:
    # same as `type(mapping) is object`
    if isinstance(mapping, op.JustObject):
        return value

    return mapping[value]


intmap(1)  # ok
intmap(1, {1: 42})  # ok
intmap(1, "some object")  # rejected
```

> [!TIP]
> The `Just{Bytes,Int,Float,Complex,Date,Object}` protocols are runtime-checkable,
> so that `instance(42, JustInt) is True` and `instance(bool(), JustInt) is False`.
> It's implemented through meta-classes, and type-checkers have no problem with it.

| `optype` type | accepts instances of |
| ------------- | -------------------- |
| `Just[T]`     | `T`                  |
| `JustInt`     | `builtins.int`       |
| `JustFloat`   | `builtins.float`     |
| `JustComplex` | `builtins.complex`   |
| `JustBytes`   | `builtins.bytes`     |
| `JustObject`  | `builtins.object`    |
| `JustDate`    | `datetime.date`      |

#### Builtin type conversion

The return type of these special methods is *invariant*. Python will raise an
error if some other (sub)type is returned.
This is why these `optype` interfaces don't accept generic type arguments.

<table>
    <tr>
        <th colspan="3" align="center">operator</th>
        <th colspan="2" align="center">operand</th>
    </tr>
    <tr>
        <td>expression</td>
        <th>function</th>
        <th>type</th>
        <td>method</td>
        <th>type</th>
    </tr>
    <tr>
        <td><code>complex(_)</code></td>
        <td><code>do_complex</code></td>
        <td><code>DoesComplex</code></td>
        <td><code>__complex__</code></td>
        <td><code>CanComplex</code></td>
    </tr>
    <tr>
        <td><code>float(_)</code></td>
        <td><code>do_float</code></td>
        <td><code>DoesFloat</code></td>
        <td><code>__float__</code></td>
        <td><code>CanFloat</code></td>
    </tr>
    <tr>
        <td><code>int(_)</code></td>
        <td><code>do_int</code></td>
        <td><code>DoesInt</code></td>
        <td><code>__int__</code></td>
        <td><code>CanInt[+R: int = int]</code></td>
    </tr>
    <tr>
        <td><code>bool(_)</code></td>
        <td><code>do_bool</code></td>
        <td><code>DoesBool</code></td>
        <td><code>__bool__</code></td>
        <td><code>CanBool[+R: bool = bool]</code></td>
    </tr>
    <tr>
        <td><code>bytes(_)</code></td>
        <td><code>do_bytes</code></td>
        <td><code>DoesBytes</code></td>
        <td><code>__bytes__</code></td>
        <td><code>CanBytes[+R: bytes = bytes]</code></td>
    </tr>
    <tr>
        <td><code>str(_)</code></td>
        <td><code>do_str</code></td>
        <td><code>DoesStr</code></td>
        <td><code>__str__</code></td>
        <td><code>CanStr[+R: str = str]</code></td>
    </tr>
</table>

> [!NOTE]
> The `Can*` interfaces of the types that can used as `typing.Literal`
> accept an optional type parameter `R`.
> This can be used to indicate a literal return type,
> for surgically precise typing, e.g. `None`, `True`, and `42` are
> instances of `CanBool[Literal[False]]`, `CanInt[Literal[1]]`, and
> `CanStr[Literal['42']]`, respectively.

These formatting methods are allowed to return instances that are a subtype
of the `str` builtin. The same holds for the `__format__` argument.
So if you're a 10x developer that wants to hack Python's f-strings, but only
if your type hints are spot-on; `optype` is you friend.

<table>
    <tr>
        <th colspan="3" align="center">operator</th>
        <th colspan="2" align="center">operand</th>
    </tr>
    <tr>
        <td>expression</td>
        <th>function</th>
        <th>type</th>
        <td>method</td>
        <th>type</th>
    </tr>
    <tr>
        <td><code>repr(_)</code></td>
        <td><code>do_repr</code></td>
        <td><code>DoesRepr</code></td>
        <td><code>__repr__</code></td>
        <td><code>CanRepr[+R:str = str]</code></td>
    </tr>
    <tr>
        <td><code>format(_, x)</code></td>
        <td><code>do_format</code></td>
        <td><code>DoesFormat</code></td>
        <td><code>__format__</code></td>
        <td><code>CanFormat[-T:str = str, +R:str = str]</code></td>
    </tr>
</table>

Additionally, `optype` provides protocols for types with (custom) *hash* or
*index* methods:

<table>
    <tr>
        <th colspan="3" align="center">operator</th>
        <th colspan="2" align="center">operand</th>
    </tr>
    <tr>
        <td>expression</td>
        <th>function</th>
        <th>type</th>
        <td>method</td>
        <th>type</th>
    </tr>
    <tr>
        <td><code>hash(_)</code></td>
        <td><code>do_hash</code></td>
        <td><code>DoesHash</code></td>
        <td><code>__hash__</code></td>
        <td><code>CanHash</code></td>
    </tr>
    <tr>
        <td>
            <code>_.__index__()</code>
            (<a href="https://docs.python.org/3/reference/datamodel.html#object.__index__">docs</a>)
        </td>
        <td><code>do_index</code></td>
        <td><code>DoesIndex</code></td>
        <td><code>__index__</code></td>
        <td><code>CanIndex[+R: int = int]</code></td>
    </tr>
</table>

#### Rich relations

The "rich" comparison special methods often return a `bool`.
However, instances of any type can be returned (e.g. a numpy array).
This is why the corresponding `optype.Can*` interfaces accept a second type
argument for the return type, that defaults to `bool` when omitted.
The first type parameter matches the passed method argument, i.e. the
right-hand side operand, denoted here as `x`.

<table>
    <tr>
        <th colspan="4" align="center">operator</th>
        <th colspan="2" align="center">operand</th>
    </tr>
    <tr>
        <td>expression</td>
        <td>reflected</td>
        <th>function</th>
        <th>type</th>
        <td>method</td>
        <th>type</th>
    </tr>
    <tr>
        <td><code>_ == x</code></td>
        <td><code>x == _</code></td>
        <td><code>do_eq</code></td>
        <td><code>DoesEq</code></td>
        <td><code>__eq__</code></td>
        <td><code>CanEq[-T = object, +R = bool]</code></td>
    </tr>
    <tr>
        <td><code>_ != x</code></td>
        <td><code>x != _</code></td>
        <td><code>do_ne</code></td>
        <td><code>DoesNe</code></td>
        <td><code>__ne__</code></td>
        <td><code>CanNe[-T = object, +R = bool]</code></td>
    </tr>
    <tr>
        <td><code>_ < x</code></td>
        <td><code>x > _</code></td>
        <td><code>do_lt</code></td>
        <td><code>DoesLt</code></td>
        <td><code>__lt__</code></td>
        <td><code>CanLt[-T, +R = bool]</code></td>
    </tr>
    <tr>
        <td><code>_ <= x</code></td>
        <td><code>x >= _</code></td>
        <td><code>do_le</code></td>
        <td><code>DoesLe</code></td>
        <td><code>__le__</code></td>
        <td><code>CanLe[-T, +R = bool]</code></td>
    </tr>
    <tr>
        <td><code>_ > x</code></td>
        <td><code>x < _</code></td>
        <td><code>do_gt</code></td>
        <td><code>DoesGt</code></td>
        <td><code>__gt__</code></td>
        <td><code>CanGt[-T, +R = bool]</code></td>
    </tr>
    <tr>
        <td><code>_ >= x</code></td>
        <td><code>x <= _</code></td>
        <td><code>do_ge</code></td>
        <td><code>DoesGe</code></td>
        <td><code>__ge__</code></td>
        <td><code>CanGe[-T, +R = bool]</code></td>
    </tr>
</table>

#### Binary operations

In the [Python docs][NT], these are referred to as "arithmetic operations".
But the operands aren't limited to numeric types, and because the
operations aren't required to be commutative, might be non-deterministic, and
could have side-effects.
Classifying them "arithmetic" is, at the very least, a bit of a stretch.

<table>
    <tr>
        <th colspan="3" align="center">operator</th>
        <th colspan="2" align="center">operand</th>
    </tr>
    <tr>
        <td>expression</td>
        <th>function</th>
        <th>type</th>
        <td>method</td>
        <th>type</th>
    </tr>
    <tr>
        <td><code>_ + x</code></td>
        <td><code>do_add</code></td>
        <td><code>DoesAdd</code></td>
        <td><code>__add__</code></td>
        <td>
            <code>CanAdd[-T, +R = T]</code><br>
            <code>CanAddSelf[-T]</code><br>
            <code>CanAddSame[-T?, +R?]</code>
        </td>
    </tr>
    <tr>
        <td><code>_ - x</code></td>
        <td><code>do_sub</code></td>
        <td><code>DoesSub</code></td>
        <td><code>__sub__</code></td>
        <td>
            <code>CanSub[-T, +R = T]</code><br>
            <code>CanSubSelf[-T]</code><br>
            <code>CanSubSame[-T?, +R?]</code>
        </td>
    </tr>
    <tr>
        <td><code>_ * x</code></td>
        <td><code>do_mul</code></td>
        <td><code>DoesMul</code></td>
        <td><code>__mul__</code></td>
        <td>
            <code>CanMul[-T, +R = T]</code><br>
            <code>CanMulSelf[-T]</code><br>
            <code>CanMulSame[-T?, +R?]</code>
        </td>
    </tr>
    <tr>
        <td><code>_ @ x</code></td>
        <td><code>do_matmul</code></td>
        <td><code>DoesMatmul</code></td>
        <td><code>__matmul__</code></td>
        <td>
            <code>CanMatmul[-T, +R = T]</code><br>
            <code>CanMatmulSelf[-T]</code><br>
            <code>CanMatmulSame[-T?, +R?]</code>
        </td>
    </tr>
    <tr>
        <td><code>_ / x</code></td>
        <td><code>do_truediv</code></td>
        <td><code>DoesTruediv</code></td>
        <td><code>__truediv__</code></td>
        <td>
            <code>CanTruediv[-T, +R = T]</code><br>
            <code>CanTruedivSelf[-T]</code><br>
            <code>CanTruedivSame[-T?, +R?]</code>
        </td>
    </tr>
    <tr>
        <td><code>_ // x</code></td>
        <td><code>do_floordiv</code></td>
        <td><code>DoesFloordiv</code></td>
        <td><code>__floordiv__</code></td>
        <td>
            <code>CanFloordiv[-T, +R = T]</code><br>
            <code>CanFloordivSelf[-T]</code><br>
            <code>CanFloordivSame[-T?, +R?]</code>
        </td>
    </tr>
    <tr>
        <td><code>_ % x</code></td>
        <td><code>do_mod</code></td>
        <td><code>DoesMod</code></td>
        <td><code>__mod__</code></td>
        <td>
            <code>CanMod[-T, +R = T]</code><br>
            <code>CanModSelf[-T]</code><br>
            <code>CanModSame[-T?, +R?]</code>
        </td>
    </tr>
    <tr>
        <td><code>divmod(_, x)</code></td>
        <td><code>do_divmod</code></td>
        <td><code>DoesDivmod</code></td>
        <td><code>__divmod__</code></td>
        <td><code>CanDivmod[-T, +R]</code></td>
    </tr>
    <tr>
        <td>
            <code>_ ** x</code><br/>
            <code>pow(_, x)</code>
        </td>
        <td><code>do_pow/2</code></td>
        <td><code>DoesPow</code></td>
        <td><code>__pow__</code></td>
        <td>
            <code>CanPow2[-T, +R = T]</code><br>
            <code>CanPowSelf[-T]</code><br>
            <code>CanPowSame[-T?, +R?]</code>
        </td>
    </tr>
    <tr>
        <td><code>pow(_, x, m)</code></td>
        <td><code>do_pow/3</code></td>
        <td><code>DoesPow</code></td>
        <td><code>__pow__</code></td>
        <td><code>CanPow3[-T, -M, +R = int]</code></td>
    </tr>
    <tr>
        <td><code>_ << x</code></td>
        <td><code>do_lshift</code></td>
        <td><code>DoesLshift</code></td>
        <td><code>__lshift__</code></td>
        <td>
            <code>CanLshift[-T, +R = T]</code><br>
            <code>CanLshiftSelf[-T]</code><br>
            <code>CanLshiftSame[-T?, +R?]</code>
        </td>
    </tr>
    <tr>
        <td><code>_ >> x</code></td>
        <td><code>do_rshift</code></td>
        <td><code>DoesRshift</code></td>
        <td><code>__rshift__</code></td>
        <td>
            <code>CanRshift[-T, +R = T]</code><br>
            <code>CanRshiftSelf[-T]</code><br>
            <code>CanRshiftSame[-T?, +R?]</code>
        </td>
    </tr>
    <tr>
        <td><code>_ & x</code></td>
        <td><code>do_and</code></td>
        <td><code>DoesAnd</code></td>
        <td><code>__and__</code></td>
        <td>
            <code>CanAnd[-T, +R = T]</code><br>
            <code>CanAndSelf[-T]</code><br>
            <code>CanAndSame[-T?, +R?]</code>
        </td>
    </tr>
    <tr>
        <td><code>_ ^ x</code></td>
        <td><code>do_xor</code></td>
        <td><code>DoesXor</code></td>
        <td><code>__xor__</code></td>
        <td>
            <code>CanXor[-T, +R = T]</code><br>
            <code>CanXorSelf[-T]</code><br>
            <code>CanXorSame[-T?, +R?]</code>
        </td>
    </tr>
    <tr>
        <td><code>_ | x</code></td>
        <td><code>do_or</code></td>
        <td><code>DoesOr</code></td>
        <td><code>__or__</code></td>
        <td>
            <code>CanOr[-T, +R = T]</code><br>
            <code>CanOrSelf[-T]</code><br>
            <code>CanOrSame[-T?, +R?]</code>
        </td>
    </tr>
</table>

> [!TIP]
> Because `pow()` can take an optional third argument, `optype`
> provides separate interfaces for `pow()` with two and three arguments.
> Additionally, there is the overloaded intersection type
> `type CanPow[-T, -M, +R, +RM] = CanPow2[T, R] & CanPow3[T, M, RM]`, as interface
> for types that can take an optional third argument.

<!-- markdownlints needs this -->

> [!NOTE]
> The `Can*Self` protocols method return `typing.Self` and optionally accept `T` and
> `R`. The `Can*Same` protocols also return `Self`, but instead accept `Self | T`, with
> `T` and `R` optional generic type parameters that default to `typing.Never`.
> To illustrate, `CanAddSelf[T]` implements `__add__` as `(self, rhs: T, /) -> Self`,
> while `CanAddSame[T, R]` implements it as `(self, rhs: Self | T, /) -> Self | R`, and
> `CanAddSame` (without `T` and `R`) as `(self, rhs: Self, /) -> Self`.

#### Reflected operations

For the binary infix operators above, `optype` additionally provides
interfaces with *reflected* (swapped) operands, e.g. `__radd__` is a reflected
`__add__`.
They are named like the original, but prefixed with `CanR` prefix, i.e.
`__name__.replace('Can', 'CanR')`.

<table>
    <tr>
        <th colspan="3" align="center">operator</th>
        <th colspan="2" align="center">operand</th>
    </tr>
    <tr>
        <td>expression</td>
        <th>function</th>
        <th>type</th>
        <td>method</td>
        <th>type</th>
    </tr>
    <tr>
        <td><code>x + _</code></td>
        <td><code>do_radd</code></td>
        <td><code>DoesRAdd</code></td>
        <td><code>__radd__</code></td>
        <td>
            <code>CanRAdd[-T, +R=T]</code><br>
            <code>CanRAddSelf[-T]</code>
        </td>
    </tr>
    <tr>
        <td><code>x - _</code></td>
        <td><code>do_rsub</code></td>
        <td><code>DoesRSub</code></td>
        <td><code>__rsub__</code></td>
        <td>
            <code>CanRSub[-T, +R=T]</code><br>
            <code>CanRSubSelf[-T]</code>
        </td>
    </tr>
    <tr>
        <td><code>x * _</code></td>
        <td><code>do_rmul</code></td>
        <td><code>DoesRMul</code></td>
        <td><code>__rmul__</code></td>
        <td>
            <code>CanRMul[-T, +R=T]</code><br>
            <code>CanRMulSelf[-T]</code>
        </td>
    </tr>
    <tr>
        <td><code>x @ _</code></td>
        <td><code>do_rmatmul</code></td>
        <td><code>DoesRMatmul</code></td>
        <td><code>__rmatmul__</code></td>
        <td>
            <code>CanRMatmul[-T, +R=T]</code><br>
            <code>CanRMatmulSelf[-T]</code>
        </td>
    </tr>
    <tr>
        <td><code>x / _</code></td>
        <td><code>do_rtruediv</code></td>
        <td><code>DoesRTruediv</code></td>
        <td><code>__rtruediv__</code></td>
        <td>
            <code>CanRTruediv[-T, +R=T]</code><br>
            <code>CanRTruedivSelf[-T]</code>
        </td>
    </tr>
    <tr>
        <td><code>x // _</code></td>
        <td><code>do_rfloordiv</code></td>
        <td><code>DoesRFloordiv</code></td>
        <td><code>__rfloordiv__</code></td>
        <td>
            <code>CanRFloordiv[-T, +R=T]</code><br>
            <code>CanRFloordivSelf[-T]</code>
        </td>
    </tr>
    <tr>
        <td><code>x % _</code></td>
        <td><code>do_rmod</code></td>
        <td><code>DoesRMod</code></td>
        <td><code>__rmod__</code></td>
        <td>
            <code>CanRMod[-T, +R=T]</code><br>
            <code>CanRModSelf[-T]</code>
        </td>
    </tr>
    <tr>
        <td><code>divmod(x, _)</code></td>
        <td><code>do_rdivmod</code></td>
        <td><code>DoesRDivmod</code></td>
        <td><code>__rdivmod__</code></td>
        <td><code>CanRDivmod[-T, +R]</code></td>
    </tr>
    <tr>
        <td>
            <code>x ** _</code><br/>
            <code>pow(x, _)</code>
        </td>
        <td><code>do_rpow</code></td>
        <td><code>DoesRPow</code></td>
        <td><code>__rpow__</code></td>
        <td>
            <code>CanRPow[-T, +R=T]</code><br>
            <code>CanRPowSelf[-T]</code>
        </td>
    </tr>
    <tr>
        <td><code>x << _</code></td>
        <td><code>do_rlshift</code></td>
        <td><code>DoesRLshift</code></td>
        <td><code>__rlshift__</code></td>
        <td>
            <code>CanRLshift[-T, +R=T]</code><br>
            <code>CanRLshiftSelf[-T]</code>
        </td>
    </tr>
    <tr>
        <td><code>x >> _</code></td>
        <td><code>do_rrshift</code></td>
        <td><code>DoesRRshift</code></td>
        <td><code>__rrshift__</code></td>
        <td>
            <code>CanRRshift[-T, +R=T]</code><br>
            <code>CanRRshiftSelf[-T]</code>
        </td>
    </tr>
    <tr>
        <td><code>x & _</code></td>
        <td><code>do_rand</code></td>
        <td><code>DoesRAnd</code></td>
        <td><code>__rand__</code></td>
        <td>
            <code>CanRAnd[-T, +R=T]</code><br>
            <code>CanRAndSelf[-T]</code>
        </td>
    </tr>
    <tr>
        <td><code>x ^ _</code></td>
        <td><code>do_rxor</code></td>
        <td><code>DoesRXor</code></td>
        <td><code>__rxor__</code></td>
        <td>
            <code>CanRXor[-T, +R=T]</code><br>
            <code>CanRXorSelf[-T]</code>
        </td>
    </tr>
    <tr>
        <td><code>x | _</code></td>
        <td><code>do_ror</code></td>
        <td><code>DoesROr</code></td>
        <td><code>__ror__</code></td>
        <td>
            <code>CanROr[-T, +R=T]</code><br>
            <code>CanROrSelf[-T]</code>
        </td>
    </tr>
</table>

> [!NOTE]
> `CanRPow` corresponds to `CanPow2`; the 3-parameter "modulo" `pow` does not
> reflect in Python.
>
> According to the relevant [python docs][RPOW]:
>
>> Note that ternary `pow()` will not try calling `__rpow__()` (the coercion
>> rules would become too complicated).

[RPOW]: https://docs.python.org/3/reference/datamodel.html#object.__rpow__

#### Inplace operations

Similar to the reflected ops, the inplace/augmented ops are prefixed with
`CanI`, namely:

<table>
    <tr>
        <th colspan="3" align="center">operator</th>
        <th colspan="2" align="center">operand</th>
    </tr>
    <tr>
        <td>expression</td>
        <th>function</th>
        <th>type</th>
        <td>method</td>
        <th>types</th>
    </tr>
    <tr>
        <td><code>_ += x</code></td>
        <td><code>do_iadd</code></td>
        <td><code>DoesIAdd</code></td>
        <td><code>__iadd__</code></td>
        <td>
            <code>CanIAdd[-T, +R]</code><br>
            <code>CanIAddSelf[-T]</code><br>
            <code>CanIAddSame[-T?]</code>
        </td>
    </tr>
    <tr>
        <td><code>_ -= x</code></td>
        <td><code>do_isub</code></td>
        <td><code>DoesISub</code></td>
        <td><code>__isub__</code></td>
        <td>
            <code>CanISub[-T, +R]</code><br>
            <code>CanISubSelf[-T]</code><br>
            <code>CanISubSame[-T?]</code>
        </td>
    </tr>
    <tr>
        <td><code>_ *= x</code></td>
        <td><code>do_imul</code></td>
        <td><code>DoesIMul</code></td>
        <td><code>__imul__</code></td>
        <td>
            <code>CanIMul[-T, +R]</code><br>
            <code>CanIMulSelf[-T]</code><br>
            <code>CanIMulSame[-T?]</code>
        </td>
    </tr>
    <tr>
        <td><code>_ @= x</code></td>
        <td><code>do_imatmul</code></td>
        <td><code>DoesIMatmul</code></td>
        <td><code>__imatmul__</code></td>
        <td>
            <code>CanIMatmul[-T, +R]</code><br>
            <code>CanIMatmulSelf[-T]</code><br>
            <code>CanIMatmulSame[-T?]</code>
        </td>
    </tr>
    <tr>
        <td><code>_ /= x</code></td>
        <td><code>do_itruediv</code></td>
        <td><code>DoesITruediv</code></td>
        <td><code>__itruediv__</code></td>
        <td>
            <code>CanITruediv[-T, +R]</code><br>
            <code>CanITruedivSelf[-T]</code><br>
            <code>CanITruedivSame[-T?]</code>
        </td>
    </tr>
    <tr>
        <td><code>_ //= x</code></td>
        <td><code>do_ifloordiv</code></td>
        <td><code>DoesIFloordiv</code></td>
        <td><code>__ifloordiv__</code></td>
        <td>
            <code>CanIFloordiv[-T, +R]</code><br>
            <code>CanIFloordivSelf[-T]</code><br>
            <code>CanIFloordivSame[-T?]</code>
        </td>
    </tr>
    <tr>
        <td><code>_ %= x</code></td>
        <td><code>do_imod</code></td>
        <td><code>DoesIMod</code></td>
        <td><code>__imod__</code></td>
        <td>
            <code>CanIMod[-T, +R]</code><br>
            <code>CanIModSelf[-T]</code><br>
            <code>CanIModSame[-T?]</code>
        </td>
    </tr>
    <tr>
        <td><code>_ **= x</code></td>
        <td><code>do_ipow</code></td>
        <td><code>DoesIPow</code></td>
        <td><code>__ipow__</code></td>
        <td>
            <code>CanIPow[-T, +R]</code><br>
            <code>CanIPowSelf[-T]</code><br>
            <code>CanIPowSame[-T?]</code>
        </td>
    </tr>
    <tr>
        <td><code>_ <<= x</code></td>
        <td><code>do_ilshift</code></td>
        <td><code>DoesILshift</code></td>
        <td><code>__ilshift__</code></td>
        <td>
            <code>CanILshift[-T, +R]</code><br>
            <code>CanILshiftSelf[-T]</code><br>
            <code>CanILshiftSame[-T?]</code>
        </td>
    </tr>
    <tr>
        <td><code>_ >>= x</code></td>
        <td><code>do_irshift</code></td>
        <td><code>DoesIRshift</code></td>
        <td><code>__irshift__</code></td>
        <td>
            <code>CanIRshift[-T, +R]</code><br>
            <code>CanIRshiftSelf[-T]</code><br>
            <code>CanIRshiftSame[-T?]</code>
        </td>
    </tr>
    <tr>
        <td><code>_ &= x</code></td>
        <td><code>do_iand</code></td>
        <td><code>DoesIAnd</code></td>
        <td><code>__iand__</code></td>
        <td>
            <code>CanIAnd[-T, +R]</code><br>
            <code>CanIAndSelf[-T]</code><br>
            <code>CanIAndSame[-T?]</code>
        </td>
    </tr>
    <tr>
        <td><code>_ ^= x</code></td>
        <td><code>do_ixor</code></td>
        <td><code>DoesIXor</code></td>
        <td><code>__ixor__</code></td>
        <td>
            <code>CanIXor[-T, +R]</code><br>
            <code>CanIXorSelf[-T]</code><br>
            <code>CanIXorSame[-T?]</code>
        </td>
    </tr>
    <tr>
        <td><code>_ |= x</code></td>
        <td><code>do_ior</code></td>
        <td><code>DoesIOr</code></td>
        <td><code>__ior__</code></td>
        <td>
            <code>CanIOr[-T, +R]</code><br>
            <code>CanIOrSelf[-T]</code><br>
            <code>CanIOrSame[-T?]</code>
        </td>
    </tr>
</table>

These inplace operators usually return themselves (after some in-place mutation).
But unfortunately, it currently isn't possible to use `Self` for this (i.e.
something like `type MyAlias[T] = optype.CanIAdd[T, Self]` isn't allowed).
So to help ease this unbearable pain, `optype` comes equipped with ready-made
aliases for you to use. They bear the same name, with an additional `*Self`
suffix, e.g. `optype.CanIAddSelf[T]`.

> [!NOTE]
> The `CanI*Self` protocols method return `typing.Self` and optionally accept `T`. The
> `CanI*Same` protocols also return `Self`, but instead accept `rhs: Self | T`. Since
> `T` defaults to `Never`, it will accept `rhs: Self | Never` if `T` is not provided,
> which is equivalent to `rhs: Self`.
>
> *Available since `0.12.1`*

#### Unary operations

<table>
    <tr>
        <th colspan="3" align="center">operator</th>
        <th colspan="2" align="center">operand</th>
    </tr>
    <tr>
        <td>expression</td>
        <th>function</th>
        <th>type</th>
        <td>method</td>
        <th>types</th>
    </tr>
    <tr>
        <td><code>+_</code></td>
        <td><code>do_pos</code></td>
        <td><code>DoesPos</code></td>
        <td><code>__pos__</code></td>
        <td>
            <code>CanPos[+R]</code><br>
            <code>CanPosSelf[+R?]</code>
        </td>
    </tr>
    <tr>
        <td><code>-_</code></td>
        <td><code>do_neg</code></td>
        <td><code>DoesNeg</code></td>
        <td><code>__neg__</code></td>
        <td>
            <code>CanNeg[+R]</code><br>
            <code>CanNegSelf[+R?]</code>
        </td>
    </tr>
    <tr>
        <td><code>~_</code></td>
        <td><code>do_invert</code></td>
        <td><code>DoesInvert</code></td>
        <td><code>__invert__</code></td>
        <td>
            <code>CanInvert[+R]</code><br>
            <code>CanInvertSelf[+R?]</code>
        </td>
    </tr>
    <tr>
        <td><code>abs(_)</code></td>
        <td><code>do_abs</code></td>
        <td><code>DoesAbs</code></td>
        <td><code>__abs__</code></td>
        <td>
            <code>CanAbs[+R]</code><br>
            <code>CanAbsSelf[+R?]</code>
        </td>
    </tr>
</table>

The `Can*Self` variants return `-> Self` instead of `R`. Since optype 0.12.1 these
also accept an optional `R` type parameter (with a default of `Never`), which, when
provided, will result in a return type of `-> Self | R`.

#### Rounding

The `round()` built-in function takes an optional second argument.
From a typing perspective, `round()` has two overloads, one with 1 parameter,
and one with two.
For both overloads, `optype` provides separate operand interfaces:
`CanRound1[R]` and `CanRound2[T, RT]`.
Additionally, `optype` also provides their (overloaded) intersection type:
`CanRound[-T, +R1, +R2] = CanRound1[R1] & CanRound2[T, R2]`.

<table>
    <tr>
        <th colspan="3" align="center">operator</th>
        <th colspan="2" align="center">operand</th>
    </tr>
    <tr>
        <td>expression</td>
        <th>function</th>
        <th>type</th>
        <td>method</td>
        <th>type</th>
    </tr>
    <tr>
        <td><code>round(_)</code></td>
        <td><code>do_round/1</code></td>
        <td><code>DoesRound</code></td>
        <td><code>__round__/1</code></td>
        <td><code>CanRound1[+R=int]</code><br/></td>
    </tr>
    <tr>
        <td><code>round(_, n)</code></td>
        <td><code>do_round/2</code></td>
        <td><code>DoesRound</code></td>
        <td><code>__round__/2</code></td>
        <td><code>CanRound2[-T=int, +R=float]</code><br/></td>
    </tr>
    <tr>
        <td><code>round(_, n=...)</code></td>
        <td><code>do_round</code></td>
        <td><code>DoesRound</code></td>
        <td><code>__round__</code></td>
        <td><code>CanRound[-T=int, +R1=int, +R2=float]</code></td>
    </tr>
</table>

For example, type-checkers will mark the following code as valid (tested with
pyright in strict mode):

```python
x: float = 3.14
x1: CanRound1[int] = x
x2: CanRound2[int, float] = x
x3: CanRound[int, int, float] = x
```

Furthermore, there are the alternative rounding functions from the
[`math`][MATH] standard library:

<table>
    <tr>
        <th colspan="3" align="center">operator</th>
        <th colspan="2" align="center">operand</th>
    </tr>
    <tr>
        <td>expression</td>
        <th>function</th>
        <th>type</th>
        <td>method</td>
        <th>type</th>
    </tr>
    <tr>
        <td><code>math.trunc(_)</code></td>
        <td><code>do_trunc</code></td>
        <td><code>DoesTrunc</code></td>
        <td><code>__trunc__</code></td>
        <td><code>CanTrunc[+R=int]</code></td>
    </tr>
    <tr>
        <td><code>math.floor(_)</code></td>
        <td><code>do_floor</code></td>
        <td><code>DoesFloor</code></td>
        <td><code>__floor__</code></td>
        <td><code>CanFloor[+R=int]</code></td>
    </tr>
    <tr>
        <td><code>math.ceil(_)</code></td>
        <td><code>do_ceil</code></td>
        <td><code>DoesCeil</code></td>
        <td><code>__ceil__</code></td>
        <td><code>CanCeil[+R=int]</code></td>
    </tr>
</table>

Almost all implementations use `int` for `R`.
In fact, if no type for `R` is specified, it will default in `int`.
But technically speaking, these methods can be made to return anything.

[MATH]: https://docs.python.org/3/library/math.html
[NT]: https://docs.python.org/3/reference/datamodel.html#emulating-numeric-types

#### Callables

Unlike `operator`, `optype` provides an operator for callable objects:
`optype.do_call(f, *args. **kwargs)`.

`CanCall` is similar to `collections.abc.Callable`, but is runtime-checkable,
and doesn't use esoteric hacks.

<table>
    <tr>
        <th colspan="3" align="center">operator</th>
        <th colspan="2" align="center">operand</th>
    </tr>
    <tr>
        <td>expression</td>
        <th>function</th>
        <th>type</th>
        <td>method</td>
        <th>type</th>
    </tr>
    <tr>
        <td><code>_(*args, **kwargs)</code></td>
        <td><code>do_call</code></td>
        <td><code>DoesCall</code></td>
        <td><code>__call__</code></td>
        <td><code>CanCall[**Tss, +R]</code></td>
    </tr>
</table>

> [!NOTE]
> Pyright (and probably other typecheckers) tend to accept
> `collections.abc.Callable` in more places than `optype.CanCall`.
> This could be related to the lack of co/contra-variance specification for
> `typing.ParamSpec` (they should almost always be contravariant, but
> currently they can only be invariant).
>
> In case you encounter such a situation, please open an issue about it, so we
> can investigate further.

#### Iteration

The operand `x` of `iter(_)` is within Python known as an *iterable*, which is
what `collections.abc.Iterable[V]` is often used for (e.g. as base class, or
for instance checking).

The `optype` analogue is `CanIter[R]`, which as the name suggests,
also implements `__iter__`. But unlike `Iterable[V]`, its type parameter `R`
binds to the return type of `iter(_) -> R`. This makes it possible to annotate
the specific type of the *iterable* that `iter(_)` returns. `Iterable[V]` is
only able to annotate the type of the iterated value. To see why that isn't
possible, see [python/typing#548](https://github.com/python/typing/issues/548).

The `collections.abc.Iterator[V]` is even more awkward; it is a subtype of
`Iterable[V]`. For those familiar with `collections.abc` this might come as a
surprise, but an iterator only needs to implement `__next__`, `__iter__` isn't
needed. This means that the `Iterator[V]` is unnecessarily restrictive.
Apart from that being theoretically "ugly", it has significant performance
implications, because the time-complexity of `isinstance` on a
`typing.Protocol` is $O(n)$, with the $n$ referring to the amount of members.
So even if the overhead of the inheritance and the `abc.ABC` usage is ignored,
`collections.abc.Iterator` is twice as slow as it needs to be.

That's one of the (many) reasons that `optype.CanNext[V]` and
`optype.CanIter[R]` are the better alternatives to `Iterable` and `Iterator`
from the abracadabra collections. This is how they are defined:

<table>
    <tr>
        <th colspan="3" align="center">operator</th>
        <th colspan="2" align="center">operand</th>
    </tr>
    <tr>
        <td>expression</td>
        <th>function</th>
        <th>type</th>
        <td>method</td>
        <th>type</th>
    </tr>
    <tr>
        <td><code>next(_)</code></td>
        <td><code>do_next</code></td>
        <td><code>DoesNext</code></td>
        <td><code>__next__</code></td>
        <td><code>CanNext[+V]</code></td>
    </tr>
    <tr>
        <td><code>iter(_)</code></td>
        <td><code>do_iter</code></td>
        <td><code>DoesIter</code></td>
        <td><code>__iter__</code></td>
        <td><code>CanIter[+R: CanNext[object]]</code></td>
    </tr>
</table>

For the sake of compatibility with `collections.abc`, there is
`optype.CanIterSelf[V]`, which is a protocol whose `__iter__` returns
`typing.Self`, as well as a `__next__` method that returns `T`.
I.e. it is equivalent to `collections.abc.Iterator[V]`, but without the `abc`
nonsense.

#### Awaitables

The `optype.CanAwait[R]` is almost the same as `collections.abc.Awaitable[R]`, except
that `optype.CanAwait[R]` is a pure interface, whereas `Awaitable` is
also an abstract base class (making it absolutely useless when writing stubs).

<table>
    <tr>
        <th align="center">operator</th>
        <th colspan="2" align="center">operand</th>
    </tr>
    <tr>
        <td>expression</td>
        <td>method</td>
        <th>type</th>
    </tr>
    <tr>
        <td><code>await _</code></td>
        <td><code>__await__</code></td>
        <td><code>CanAwait[+R]</code></td>
    </tr>
</table>

#### Async Iteration

Yes, you guessed it right; the abracadabra collections made the exact same
mistakes for the async iterablors (or was it "iteramblers"...?).

But fret not; the `optype` alternatives are right here:

<table>
    <tr>
        <th colspan="3" align="center">operator</th>
        <th colspan="2" align="center">operand</th>
    </tr>
    <tr>
        <td>expression</td>
        <th>function</th>
        <th>type</th>
        <td>method</td>
        <th>type</th>
    </tr>
    <tr>
        <td><code>anext(_)</code></td>
        <td><code>do_anext</code></td>
        <td><code>DoesANext</code></td>
        <td><code>__anext__</code></td>
        <td><code>CanANext[+V]</code></td>
    </tr>
    <tr>
        <td><code>aiter(_)</code></td>
        <td><code>do_aiter</code></td>
        <td><code>DoesAIter</code></td>
        <td><code>__aiter__</code></td>
        <td><code>CanAIter[+R: CanAnext[object]]</code></td>
    </tr>
</table>

But wait, shouldn't `V` be a `CanAwait`? Well, only if you don't want to get
fired...
Technically speaking, `__anext__` can return any type, and `anext` will pass
it along without nagging. For details, see the discussion at [python/typeshed#7491][AN].
Just because something is legal, doesn't mean it's a good idea (don't eat the
yellow snow).

Additionally, there is `optype.CanAIterSelf[R]`, with both the
`__aiter__() -> Self` and the `__anext__() -> V` methods.

[AN]: https://github.com/python/typeshed/pull/7491

#### Containers

<table>
    <tr>
        <th colspan="3" align="center">operator</th>
        <th colspan="2" align="center">operand</th>
    </tr>
    <tr>
        <td>expression</td>
        <th>function</th>
        <th>type</th>
        <td>method</td>
        <th>type</th>
    </tr>
    <tr>
        <td><code>len(_)</code></td>
        <td><code>do_len</code></td>
        <td><code>DoesLen</code></td>
        <td><code>__len__</code></td>
        <td><code>CanLen[+R:int=int]</code></td>
    </tr>
    <tr>
        <td>
            <code>_.__length_hint__()</code>
            (<a href="https://docs.python.org/3/reference/datamodel.html#object.__length_hint__">docs</a>)
        </td>
        <td><code>do_length_hint</code></td>
        <td><code>DoesLengthHint</code></td>
        <td><code>__length_hint__</code></td>
        <td><code>CanLengthHint[+R:int=int]</code></td>
    </tr>
    <tr>
        <td><code>_[k]</code></td>
        <td><code>do_getitem</code></td>
        <td><code>DoesGetitem</code></td>
        <td><code>__getitem__</code></td>
        <td><code>CanGetitem[-K, +V]</code></td>
    </tr>
    <tr>
        <td>
            <code>_.__missing__()</code>
            (<a href="https://docs.python.org/3/reference/datamodel.html#object.__missing__">docs</a>)
        </td>
        <td><code>do_missing</code></td>
        <td><code>DoesMissing</code></td>
        <td><code>__missing__</code></td>
        <td><code>CanMissing[-K, +D]</code></td>
    </tr>
    <tr>
        <td><code>_[k] = v</code></td>
        <td><code>do_setitem</code></td>
        <td><code>DoesSetitem</code></td>
        <td><code>__setitem__</code></td>
        <td><code>CanSetitem[-K, -V]</code></td>
    </tr>
    <tr>
        <td><code>del _[k]</code></td>
        <td><code>do_delitem</code></td>
        <td><code>DoesDelitem</code></td>
        <td><code>__delitem__</code></td>
        <td><code>CanDelitem[-K]</code></td>
    </tr>
    <tr>
        <td><code>k in _</code></td>
        <td><code>do_contains</code></td>
        <td><code>DoesContains</code></td>
        <td><code>__contains__</code></td>
        <td><code>CanContains[-K=object]</code></td>
    </tr>
    <tr>
        <td><code>reversed(_)</code></td>
        <td><code>do_reversed</code></td>
        <td><code>DoesReversed</code></td>
        <td><code>__reversed__</code></td>
        <td>
            <code>CanReversed[+R]</code>, or<br>
            <code>CanSequence[-I, +V, +N=int]</code>
        </td>
    </tr>
</table>

Because `CanMissing[K, D]` generally doesn't show itself without
`CanGetitem[K, V]` there to hold its hand, `optype` conveniently stitched them
together as `optype.CanGetMissing[K, V, D=V]`.

Similarly, there is `optype.CanSequence[K: CanIndex | slice, V]`, which is the
combination of both `CanLen` and `CanItem[I, V]`, and serves as a more
specific and flexible `collections.abc.Sequence[V]`.

#### Attributes

<table>
    <tr>
        <th colspan="3" align="center">operator</th>
        <th colspan="2" align="center">operand</th>
    </tr>
    <tr>
        <td>expression</td>
        <th>function</th>
        <th>type</th>
        <td>method</td>
        <th>type</th>
    </tr>
    <tr>
        <td>
            <code>v = _.k</code> or<br/>
            <code>v = getattr(_, k)</code>
        </td>
        <td><code>do_getattr</code></td>
        <td><code>DoesGetattr</code></td>
        <td><code>__getattr__</code></td>
        <td><code>CanGetattr[-K:str=str, +V=object]</code></td>
    </tr>
    <tr>
        <td>
            <code>_.k = v</code> or<br/>
            <code>setattr(_, k, v)</code>
        </td>
        <td><code>do_setattr</code></td>
        <td><code>DoesSetattr</code></td>
        <td><code>__setattr__</code></td>
        <td><code>CanSetattr[-K:str=str, -V=object]</code></td>
    </tr>
    <tr>
        <td>
            <code>del _.k</code> or<br/>
            <code>delattr(_, k)</code>
        </td>
        <td><code>do_delattr</code></td>
        <td><code>DoesDelattr</code></td>
        <td><code>__delattr__</code></td>
        <td><code>CanDelattr[-K:str=str]</code></td>
    </tr>
    <tr>
        <td><code>dir(_)</code></td>
        <td><code>do_dir</code></td>
        <td><code>DoesDir</code></td>
        <td><code>__dir__</code></td>
        <td><code>CanDir[+R:CanIter[CanIterSelf[str]]]</code></td>
    </tr>
</table>

#### Context managers

Support for the `with` statement.

<table>
    <tr>
        <th align="center">operator</th>
        <th colspan="2" align="center">operand</th>
    </tr>
    <tr>
        <td>expression</td>
        <td>method(s)</td>
        <th>type(s)</th>
    </tr>
    <tr>
        <td></td>
        <td><code>__enter__</code></td>
        <td>
            <code>CanEnter[+C]</code>, or
            <code>CanEnterSelf</code>
        </td>
    </tr>
    <tr>
        <td></td>
        <td><code>__exit__</code></td>
        <td>
            <code>CanExit[+R = None]</code>
        </td>
    </tr>
    <tr>
        <td><code>with _ as c:</code></td>
        <td>
            <code>__enter__</code>, and <br>
            <code>__exit__</code>
        </td>
        <td>
            <code>CanWith[+C, +R = None]</code>, or<br>
            <code>CanWithSelf[+R = None]</code>
        </td>
    </tr>
</table>

`CanEnterSelf` and `CanWithSelf` are (runtime-checkable) aliases for
`CanEnter[Self]` and `CanWith[Self, R]`, respectively.

For the `async with` statement the interfaces look very similar:

<table>
    <tr>
        <th align="center">operator</th>
        <th colspan="2" align="center">operand</th>
    </tr>
    <tr>
        <td>expression</td>
        <td>method(s)</td>
        <th>type(s)</th>
    </tr>
    <tr>
        <td></td>
        <td><code>__aenter__</code></td>
        <td>
            <code>CanAEnter[+C]</code>, or<br>
            <code>CanAEnterSelf</code>
        </td>
    </tr>
    <tr>
        <td></td>
        <td><code>__aexit__</code></td>
        <td><code>CanAExit[+R = None]</code></td>
    </tr>
    <tr>
        <td><code>async with _ as c:</code></td>
        <td>
            <code>__aenter__</code>, and<br>
            <code>__aexit__</code>
        </td>
        <td>
            <code>CanAsyncWith[+C, +R = None]</code>, or<br>
            <code>CanAsyncWithSelf[+R = None]</code>
        </td>
    </tr>
</table>

#### Descriptors

Interfaces for [descriptors](https://docs.python.org/3/howto/descriptor.html).

<table>
    <tr>
        <th align="center">operator</th>
        <th colspan="2" align="center">operand</th>
    </tr>
    <tr>
        <td>expression</td>
        <td>method</td>
        <th>type</th>
    </tr>
    <tr>
        <td>
            <code>v: V = T().d</code><br>
            <code>vt: VT = T.d</code>
        </td>
        <td><code>__get__</code></td>
        <td><code>CanGet[-T, +V, +VT=V]</code></td>
    </tr>
    <tr>
        <td>
            <code>v: V = T().d</code><br>
            <code>vt: Self = T.d</code>
        </td>
        <td><code>__get__</code></td>
        <td><code>CanGetSelf[-T, +V]</code></td>
    </tr>
    <tr>
        <td><code>T().k = v</code></td>
        <td><code>__set__</code></td>
        <td><code>CanSet[-T, -V]</code></td>
    </tr>
    <tr>
        <td><code>del T().k</code></td>
        <td><code>__delete__</code></td>
        <td><code>CanDelete[-T]</code></td>
    </tr>
    <tr>
        <td><code>class T: d = _</code></td>
        <td><code>__set_name__</code></td>
        <td><code>CanSetName[-T, -N: str = str]</code></td>
    </tr>
</table>

#### Buffer types

Interfaces for emulating buffer types using the [buffer protocol][BP].

<table>
    <tr>
        <th align="center">operator</th>
        <th colspan="2" align="center">operand</th>
    </tr>
    <tr>
        <td>expression</td>
        <td>method</td>
        <th>type</th>
    </tr>
    <tr>
        <td><code>v = memoryview(_)</code></td>
        <td><code>__buffer__</code></td>
        <td><code>CanBuffer[-T: int = int]</code></td>
    </tr>
    <tr>
        <td><code>del v</code></td>
        <td><code>__release_buffer__</code></td>
        <td><code>CanReleaseBuffer</code></td>
    </tr>
</table>

[BP]: https://docs.python.org/3/reference/datamodel.html#python-buffer-protocol

### `optype.copy`

For the [`copy`][CP] standard library, `optype.copy` provides the following
runtime-checkable interfaces:

<table>
    <tr>
        <th align="center"><code>copy</code> standard library</th>
        <th colspan="2" align="center"><code>optype.copy</code></th>
    </tr>
    <tr>
        <td>function</td>
        <th>type</th>
        <th>method</th>
    </tr>
    <tr>
        <td><code>copy.copy(_) -> R</code></td>
        <td><code>__copy__() -> R</code></td>
        <td><code>CanCopy[+R]</code></td>
    </tr>
    <tr>
        <td><code>copy.deepcopy(_, memo={}) -> R</code></td>
        <td><code>__deepcopy__(memo, /) -> R</code></td>
        <td><code>CanDeepcopy[+R]</code></td>
    </tr>
    <tr>
        <td>
            <code>copy.replace(_, /, **changes: V) -> R</code>
            <sup>[1]</sup>
        </td>
        <td><code>__replace__(**changes: V) -> R</code></td>
        <td><code>CanReplace[-V, +R]</code></td>
    </tr>
</table>

<sup>[1]</sup> *`copy.replace` requires `python>=3.13`
(but `optype.copy.CanReplace` doesn't)*

In practice, it makes sense that a copy of an instance is the same type as the
original.
But because `typing.Self` cannot be used as a type argument, this difficult
to properly type.
Instead, you can use the `optype.copy.Can{}Self` types, which are the
runtime-checkable equivalents of the following (recursive) type aliases:

```python
type CanCopySelf = CanCopy[CanCopySelf]
type CanDeepcopySelf = CanDeepcopy[CanDeepcopySelf]
type CanReplaceSelf[V] = CanReplace[V, CanReplaceSelf[V]]
```

[CP]: https://docs.python.org/3/library/copy.html

### `optype.dataclasses`

For the [`dataclasses`][DC] standard library, `optype.dataclasses` provides the
`HasDataclassFields[V: Mapping[str, Field]]` interface.
It can conveniently be used to check whether a type or instance is a
dataclass, i.e. `isinstance(obj, HasDataclassFields)`.

[DC]: https://docs.python.org/3/library/dataclasses.html

### `optype.inspect`

A collection of functions for runtime inspection of types, modules, and other
objects.

<table width="415px">
    <tr>
        <th>Function</th>
        <th>Description</th>
    </tr>
        <tr>
        <td><code>get_args(_)</code></td>
<td>

A better alternative to [`typing.get_args()`][GET_ARGS], that

- unpacks `typing.Annotated` and Python 3.12 `type _` alias types
  (i.e. `typing.TypeAliasType`),
- recursively flattens unions and nested `typing.Literal` types, and
- raises `TypeError` if not a type expression.

Return a `tuple[type | object, ...]` of type arguments or parameters.

To illustrate one of the (many) issues with `typing.get_args`:

```pycon
>>> from typing import Literal, TypeAlias, get_args
>>> Falsy: TypeAlias = Literal[None] | Literal[False, 0] | Literal["", b""]
>>> get_args(Falsy)
(typing.Literal[None], typing.Literal[False, 0], typing.Literal['', b''])
```

But this is in direct contradiction with the
[official typing documentation][LITERAL-DOCS]:

> When a Literal is parameterized with more than one value, it’s treated as
> exactly equivalent to the union of those types.
> That is, `Literal[v1, v2, v3]` is equivalent to
> `Literal[v1] | Literal[v2] | Literal[v3]`.

So this is why `optype.inspect.get_args` should be used

```pycon
>>> import optype as opt
>>> opt.inspect.get_args(Falsy)
(None, False, 0, '', b'')
```

Another issue of `typing.get_args` is with Python 3.12 `type _ = ...` aliases,
which are meant as a replacement for `_: typing.TypeAlias = ...`, and should
therefore be treated equally:

```pycon
>>> import typing
>>> import optype as opt
>>> type StringLike = str | bytes
>>> typing.get_args(StringLike)
()
>>> opt.inspect.get_args(StringLike)
(<class 'str'>, <class 'bytes'>)
```

Clearly, `typing.get_args` fails miserably here; it would have been better
if it would have raised an error, but it instead returns an empty tuple,
hiding the fact that it doesn't support the new `type _ = ...` aliases.
But luckily, `optype.inspect.get_args` doesn't have this problem, and treats
it just like it treats `typing.Alias` (and so do the other `optype.inspect`
functions).

</td>
    </tr>
    <tr>
        <td><code>get_protocol_members(_)</code></td>
<td>

A better alternative to [`typing.get_protocol_members()`][PROTO_MEM], that

- doesn't require Python 3.13 or above,
- supports [PEP 695][PEP695] `type _` alias types on Python 3.12 and above,
- unpacks unions of `typing.Literal` ...
- ... and flattens them if nested within another `typing.Literal`,
- treats `typing.Annotated[T]` as `T`, and
- raises a `TypeError` if the passed value isn't a type expression.

Returns a `frozenset[str]` with member names.

</td>
    </tr>
    <tr>
        <td><code>get_protocols(_)</code></td>
<td>

Returns a `frozenset[type]` of the public protocols within the passed module.
Pass `private=True` to also return the private protocols.

</td>
    </tr>
    <tr>
        <td><code>is_iterable(_)</code></td>
<td>

Check whether the object can be iterated over, i.e. if it can be used in a
`for` loop, without attempting to do so.
If `True` is returned, then the object is a `optype.typing.AnyIterable`
instance.

</td>
    </tr>
    <tr>
        <td><code>is_final(_)</code></td>
<td>

Check if the type, method / classmethod / staticmethod / property, is
decorated with [`@typing.final`][@FINAL].

Note that a `@property` won't be recognized unless the `@final` decorator is
placed *below* the `@property` decorator.
See the function docstring for more information.

</td>
    </tr>
    <tr>
        <td><code>is_protocol(_)</code></td>
<td>

A backport of [`typing.is_protocol`][IS_PROTO] that was added in Python 3.13,
a re-export of [`typing_extensions.is_protocol`][IS_PROTO_EXT].

</td>
    </tr>
    <tr>
        <td><code>is_runtime_protocol(_)</code></td>
<td>

Check if the type expression is a *runtime-protocol*, i.e. a
`typing.Protocol` *type*, decorated with `@typing.runtime_checkable` (also
supports `typing_extensions`).

</td>
    </tr>
    <tr>
        <td><code>is_union_type(_)</code></td>
<td>

Check if the type is a [`typing.Union`][UNION] type, e.g. `str | int`.

Unlike `isinstance(_, types.Union)`, this function also returns `True` for
unions of user-defined `Generic` or `Protocol` types (because those are
different union types for some reason).

</td>
    </tr>
    <tr>
        <td><code>is_generic_alias(_)</code></td>
<td>

Check if the type is a *subscripted* type, e.g. `list[str]` or
`optype.CanNext[int]`, but not `list`, `CanNext`.

Unlike `isinstance(_, typing.GenericAlias)`, this function also returns `True`
for user-defined `Generic` or `Protocol` types (because those are
use a different generic alias for some reason).

Even though technically `T1 | T2` is represented as `typing.Union[T1, T2]`
(which is a (special) generic alias), `is_generic_alias` will returns `False`
for such union types, because calling `T1 | T2` a subscripted type just
doesn't make much sense.

</td>
    </tr>
</table>

> [!NOTE]
> All functions in `optype.inspect` also work for Python 3.12 `type _` aliases
> (i.e. `types.TypeAliasType`) and with `typing.Annotated`.

[UNION]: https://docs.python.org/3/library/typing.html#typing.Union
[LITERAL-DOCS]: https://typing.readthedocs.io/en/latest/spec/literal.html#shortening-unions-of-literals
[@FINAL]: https://docs.python.org/3/library/typing.html#typing.Literal
[GET_ARGS]: https://docs.python.org/3/library/typing.html#typing.get_args
[IS_PROTO]: https://docs.python.org/3.13/library/typing.html#typing.is_protocol
[IS_PROTO_EXT]: https://typing-extensions.readthedocs.io/en/latest/#typing_extensions.is_protocol
[PROTO_MEM]: https://docs.python.org/3.13/library/typing.html#typing.get_protocol_members

### `optype.io`

A collection of protocols and type-aliases that, unlike their analogues in `_typeshed`,
are accessible at runtime, and use a consistent naming scheme.

<table>
    <tr>
        <th><code>optype.io</code> protocol</th>
        <th>implements</th>
        <th>replaces</th>
    </tr>
    <tr>
        <td><code>CanFSPath[+T: str | bytes =]</code></td>
        <td><code>__fspath__: () -> T</code></td>
        <td><code>os.PathLike[AnyStr: (str, bytes)]</code></td>
    </tr>
    <tr>
        <td><code>CanRead[+T]</code></td>
        <td><code>read: () -> T</code></td>
        <td></td>
    </tr>
    <tr>
        <td><code>CanReadN[+T]</code></td>
        <td><code>read: (int) -> T</code></td>
        <td><code>_typeshed.SupportsRead[+T]</code></td>
    </tr>
    <tr>
        <td><code>CanReadline[+T]</code></td>
        <td><code>readline: () -> T</code></td>
        <td><code>_typeshed.SupportsNoArgReadline[+T]</code></td>
    </tr>
    <tr>
        <td><code>CanReadlineN[+T]</code></td>
        <td><code>readline: (int) -> T</code></td>
        <td><code>_typeshed.SupportsReadline[+T]</code></td>
    </tr>
    <tr>
        <td><code>CanWrite[-T, +RT = object]</code></td>
        <td><code>write: (T) -> RT</code></td>
        <td><code>_typeshed.SupportsWrite[-T]</code></td>
    </tr>
    <tr>
        <td><code>CanFlush[+RT = object]</code></td>
        <td><code>flush: () -> RT</code></td>
        <td><code>_typeshed.SupportsFlush</code></td>
    </tr>
    <tr>
        <td><code>CanFileno</code></td>
        <td><code>fileno: () -> int</code></td>
        <td><code>_typeshed.HasFileno</code></td>
    </tr>
</table>

<table>
    <tr>
        <th><code>optype.io</code> type alias</th>
        <th>expression</th>
        <th>replaces</th>
    </tr>
    <tr>
        <td><code>ToPath[+T: str | bytes =]</code></td>
        <td><code>T | CanFSPath[T]</code></td>
        <td>
            <code>_typeshed.StrPath</code><br>
            <code>_typeshed.BytesPath</code><br>
            <code>_typeshed.StrOrBytesPath</code><br>
            <code>_typeshed.GenericPath[AnyStr]</code><br>
        </td>
    </tr>
    <tr>
        <td><code>ToFileno</code></td>
        <td><code>int | CanFileno</code></td>
        <td><code>_typeshed.FileDescriptorLike</code></td>
    </tr>
</table>

### `optype.json`

Type aliases for the `json` standard library:

<table>
    <tr>
        <td><code>Value</code></td>
        <td><code>AnyValue</code></td>
    </tr>
    <tr>
        <th><code>json.load(s)</code> return type</th>
        <th><code>json.dumps(s)</code> input type</th>
    </tr>
    <tr>
        <td><code>Array[V: Value = Value]</code></td>
        <td><code>AnyArray[~V: AnyValue = AnyValue]</code></td>
    </tr>
    <tr>
        <td><code>Object[V: Value = Value]</code></td>
        <td><code>AnyObject[~V: AnyValue = AnyValue]</code></td>
    </tr>
</table>

The `(Any)Value` can be any json input, i.e. `Value | Array | Object` is
equivalent to `Value`.
It's also worth noting that `Value` is a subtype of `AnyValue`, which means
that `AnyValue | Value` is equivalent to `AnyValue`.

### `optype.pickle`

For the [`pickle`][PK] standard library, `optype.pickle` provides the following
interfaces:

[PK]: https://docs.python.org/3/library/pickle.html

<table>
    <tr>
        <th>method(s)</th>
        <th>signature (bound)</th>
        <th>type</th>
    </tr>
    <tr>
        <td><code>__reduce__</code></td>
        <td><code>() -> R</code></td>
        <td><code>CanReduce[+R: str | tuple =]</code></td>
    </tr>
    <tr>
        <td><code>__reduce_ex__</code></td>
        <td><code>(CanIndex) -> R</code></td>
        <td><code>CanReduceEx[+R: str | tuple =]</code></td>
    </tr>
    <tr>
        <td><code>__getstate__</code></td>
        <td><code>() -> S</code></td>
        <td><code>CanGetstate[+S]</code></td>
    </tr>
    <tr>
        <td><code>__setstate__</code></td>
        <td><code>(S) -> None</code></td>
        <td><code>CanSetstate[-S]</code></td>
    </tr>
    <tr>
        <td>
            <code>__getnewargs__</code><br>
            <code>__new__</code>
        </td>
        <td>
            <code>() -> tuple[V, ...]</code><br>
            <code>(V) -> Self</code><br>
        </td>
        <td><code>CanGetnewargs[+V]</code></td>
    </tr>
    <tr>
        <td>
            <code>__getnewargs_ex__</code><br>
            <code>__new__</code>
        </td>
        <td>
            <code>() -> tuple[tuple[V, ...], dict[str, KV]]</code><br>
            <code>(*tuple[V, ...], **dict[str, KV]) -> Self</code><br>
        </td>
        <td><code>CanGetnewargsEx[+V, ~KV]</code></td>
    </tr>
</table>

<!--
### `optype.rick`

I'm a pickle.
-->

### `optype.string`

The [`string`](https://docs.python.org/3/library/string.html) standard
library contains practical constants, but it has two issues:

- The constants contain a collection of characters, but are represented as
  a single string. This makes it practically impossible to type-hint the
  individual characters, so typeshed currently types these constants as a
  `LiteralString`.
- The names of the constants are inconsistent, and doesn't follow
  [PEP 8](https://peps.python.org/pep-0008/#constants).

So instead, `optype.string` provides an alternative interface, that is
compatible with `string`, but with slight differences:

- For each constant, there is a corresponding `Literal` type alias for
  the *individual* characters. Its name matches the name of the constant,
  but is singular instead of plural.
- Instead of a single string, `optype.string` uses a `tuple` of characters,
  so that each character has its own `typing.Literal` annotation.
  Note that this is only tested with (based)pyright / pylance, so it might
  not work with mypy (it has more bugs than it has lines of codes).
- The names of the constant are consistent with PEP 8, and use a postfix
  notation for variants, e.g. `DIGITS_HEX` instead of `hexdigits`.
- Unlike `string`, `optype.string` has a constant (and type alias) for
  binary digits `'0'` and `'1'`; `DIGITS_BIN` (and `DigitBin`). Because
  besides `oct` and `hex` functions in `builtins`, there's also the
  `builtins.bin` function.

<table>
    <tr>
        <th colspan="2"><code>string._</code></th>
        <th colspan="2"><code>optype.string._</code></th>
    </tr>
    <tr>
        <th>constant</th>
        <th>char type</th>
        <th>constant</th>
        <th>char type</th>
    </tr>
    <tr>
        <td colspan="2" align="center"><i>missing</i></td>
        <td><code>DIGITS_BIN</code></td>
        <td><code>DigitBin</code></td>
    </tr>
    <tr>
        <td><code>octdigits</code></td>
        <td rowspan="9"><code>LiteralString</code></td>
        <td><code>DIGITS_OCT</code></td>
        <td><code>DigitOct</code></td>
    </tr>
    <tr>
        <td><code>digits</code></td>
        <td><code>DIGITS</code></td>
        <td><code>Digit</code></td>
    </tr>
    <tr>
        <td><code>hexdigits</code></td>
        <td><code>DIGITS_HEX</code></td>
        <td><code>DigitHex</code></td>
    </tr>
    <tr>
        <td><code>ascii_letters</code></td>
        <td><code>LETTERS</code></td>
        <td><code>Letter</code></td>
    </tr>
    <tr>
        <td><code>ascii_lowercase</code></td>
        <td><code>LETTERS_LOWER</code></td>
        <td><code>LetterLower</code></td>
    </tr>
    <tr>
        <td><code>ascii_uppercase</code></td>
        <td><code>LETTERS_UPPER</code></td>
        <td><code>LetterUpper</code></td>
    </tr>
    <tr>
        <td><code>punctuation</code></td>
        <td><code>PUNCTUATION</code></td>
        <td><code>Punctuation</code></td>
    </tr>
    <tr>
        <td><code>whitespace</code></td>
        <td><code>WHITESPACE</code></td>
        <td><code>Whitespace</code></td>
    </tr>
    <tr>
        <td><code>printable</code></td>
        <td><code>PRINTABLE</code></td>
        <td><code>Printable</code></td>
    </tr>
</table>

Each of the `optype.string` constants is exactly the same as the corresponding
`string` constant (after concatenation / splitting), e.g.

```pycon
>>> import string
>>> import optype as opt
>>> "".join(opt.string.PRINTABLE) == string.printable
True
>>> tuple(string.printable) == opt.string.PRINTABLE
True
```

Similarly, the values within a constant's `Literal` type exactly match the
values of its constant:

```pycon
>>> import optype as opt
>>> from optype.inspect import get_args
>>> get_args(opt.string.Printable) == opt.string.PRINTABLE
True
```

The `optype.inspect.get_args` is a non-broken variant of `typing.get_args`
that correctly flattens nested literals, type-unions, and PEP 695 type aliases,
so that it matches the official typing specs.
*In other words; `typing.get_args` is yet another fundamentally broken
python-typing feature that's useless in the situations where you need it
most.*

### `optype.typing`

#### `Any*` type aliases

Type aliases for anything that can *always* be passed to
`int`, `float`, `complex`, `iter`, or `typing.Literal`

<table>
    <tr>
        <th>Python constructor</th>
        <th><code>optype.typing</code> alias</th>
    </tr>
    <tr>
        <td><code>int(_)</code></td>
        <td><code>AnyInt</code></td>
    </tr>
    <tr>
        <td><code>float(_)</code></td>
        <td><code>AnyFloat</code></td>
    </tr>
    <tr>
        <td><code>complex(_)</code></td>
        <td><code>AnyComplex</code></td>
    </tr>
    <tr>
        <td><code>iter(_)</code></td>
        <td><code>AnyIterable</code></td>
    </tr>
    <tr>
        <td><code>typing.Literal[_]</code></td>
        <td><code>AnyLiteral</code></td>
    </tr>
</table>

> [!NOTE]
> Even though *some* `str` and `bytes` can be converted to `int`, `float`,
> `complex`, most of them can't, and are therefore not included in these
> type aliases.

#### `Empty*` type aliases

These are builtin types or collections that are empty, i.e. have length 0 or
yield no elements.

<table>
    <tr>
        <th>instance</th>
        <th><code>optype.typing</code> type</th>
    </tr>
    <tr>
        <td><code>''</code></td>
        <td><code>EmptyString</code></td>
    </tr>
    <tr>
        <td><code>b''</code></td>
        <td><code>EmptyBytes</code></td>
    </tr>
    <tr>
        <td><code>()</code></td>
        <td><code>EmptyTuple</code></td>
    </tr>
    <tr>
        <td><code>[]</code></td>
        <td><code>EmptyList</code></td>
    </tr>
    <tr>
        <td><code>{}</code></td>
        <td><code>EmptyDict</code></td>
    </tr>
    <tr>
        <td><code>set()</code></td>
        <td><code>EmptySet</code></td>
    </tr>
    <tr>
        <td><code>(i for i in range(0))</code></td>
        <td><code>EmptyIterable</code></td>
    </tr>
</table>

#### Literal types

<table>
    <tr>
        <th>Literal values</th>
        <th><code>optype.typing</code> type</th>
        <th>Notes</th>
    </tr>
    <tr>
        <td><code>{False, True}</code></td>
        <td><code>LiteralFalse</code></td>
        <td>
            Similar to <code>typing.LiteralString</code>, but for
            <code>bool</code>.
        </td>
    </tr>
    <tr>
        <td><code>{0, 1, ..., 255}</code></td>
        <td><code>LiteralByte</code></td>
        <td>
            Integers in the range 0-255, that make up a <code>bytes</code>
            or <code>bytearray</code> objects.
        </td>
    </tr>
</table>

### `optype.dlpack`

A collection of low-level types for working [DLPack](DOC-DLPACK).

#### Protocols

<table>
    <tr>
        <th>type signature</th>
        <th>bound method</th>
    </tr>
    <tr>
<td>

```plain
CanDLPack[
    +T = int,
    +D: int = int,
]
```

</td>
<td>

```python
def __dlpack__(
    *,
    stream: int | None = ...,
    max_version: tuple[int, int] | None = ...,
    dl_device: tuple[T, D] | None = ...,
    copy: bool | None = ...,
) -> types.CapsuleType: ...
```

</td>
    </tr>
    <tr></tr>
    <tr>
<td>

```plain
CanDLPackDevice[
    +T = int,
    +D: int = int,
]
```

</td>
<td>

```python
def __dlpack_device__() -> tuple[T, D]: ...
```

</td>
    </tr>
</table>

The `+` prefix indicates that the type parameter is *co*variant.

#### Enums

There are also two convenient
[`IntEnum`](https://docs.python.org/3/library/enum.html#enum.IntEnum)s
in `optype.dlpack`: `DLDeviceType` for the device types, and `DLDataTypeCode` for the
internal type-codes of the `DLPack` data types.

<table>

### `optype.numpy`

Optype supports both NumPy 1 and 2.
The current minimum supported version is `1.25`,
following [NEP 29][NEP29] and [SPEC 0][SPEC0].

When using `optype.numpy`, it is recommended to install `optype` with the
`numpy` extra, ensuring version compatibility:

```shell
pip install "optype[numpy]"
```

> [!NOTE]
> For the remainder of the `optype.numpy` docs, assume that the following
> import aliases are available.
>
> ```python
> from typing import Any, Literal
> import numpy as np
> import numpy.typing as npt
> import optype.numpy as onp
> ```
>
> For the sake of brevity and readability, the [PEP 695][PEP695] and
> [PEP 696][PEP696] type parameter syntax will be used, which is supported
> since Python 3.13.

#### Shape-typing

##### Array aliases

Optype provides the generic `onp.Array` type alias for `np.ndarray`.
It is similar to `npt.NDArray`, but includes two (optional) type parameters:
one that matches the *shape type* (`ND: tuple[int, ...]`),
and one that matches the *scalar type* (`ST: np.generic`).

When we put the definitions of `npt.NDArray` and `onp.Array` side-by-side,
their differences become clear:

<table>
<tr>
<th>

`numpy.typing.NDArray`[^1]

</th>
<th>

`optype.numpy.Array`

</th>
<th>

`optype.numpy.ArrayND`

</th>
</tr>
<tr>
<td>

```python
type NDArray[
    # no shape type
    SCT: generic,  # no default
] = ndarray[Any, dtype[SCT]]
```

</td>
<td>

```python
type Array[
    NDT: (int, ...) = (int, ...),
    SCT: generic = generic,
] = ndarray[NDT, dtype[SCT]]
```

</td>
<td>

```python
type ArrayND[
    SCT: generic = generic,
    NDT: (int, ...) = (int, ...),
] = ndarray[NDT, dtype[SCT]]
```

</td>
</tr>
</table>

Additionally, there are the four `Array{0,1,2,3}D` aliases, which are
equivalent to `Array` with `tuple[()]`, `tuple[int]`, `tuple[int, int]` and
`tuple[int, int, int]` as shape-type, respectively.

[^1]: Since `numpy>=2.2` the `NDArray` alias uses `tuple[int, ...]` as shape-type
    instead of `Any`.

> [!TIP]
> Before NumPy 2.1, the shape type parameter of `ndarray` (i.e. the type of
> `ndarray.shape`) was invariant. It is therefore recommended to not use `Literal`
> within shape types on `numpy<2.1`. So with `numpy>=2.1` you can use
> `tuple[Literal[3], Literal[3]]` without problem, but with `numpy<2.1` you should use
> `tuple[int, int]` instead.
>
> See [numpy/numpy#25729](https://github.com/numpy/numpy/issues/25729) and
> [numpy/numpy#26081](https://github.com/numpy/numpy/pull/26081) for details.

In the same way as `ArrayND` for `ndarray` (shown for reference), its subtypes
`np.ma.MaskedArray` and `np.matrix` are also aliased:

<table>
<tr>
<th>

`ArrayND` (`np.ndarray`)

</th>
<th>

`MArray` (`np.ma.MaskedArray`)

</th>
<th>

`Matrix` (`np.matrix`)

</th>
</tr>
<tr>
<td>

```python
type ArrayND[
    SCT: generic = generic,
    NDT: (int, ...) = (int, ...),
] = ndarray[NDT, dtype[SCT]]
```

</td>

<td>

```python
type MArray[
    SCT: generic = generic,
    NDT: (int, ...) = (int, ...),
] = ma.MaskedArray[NDT, dtype[SCT]]
```

</td>
<td>

```python
type Matrix[
    SCT: generic = generic,
    M: int = int,
    N: int = M,
] = matrix[(M, N), dtype[SCT]]
```

</td>
</tr>
</table>

For masked arrays with specific `ndim`, you could also use one of the four
`MArray{0,1,2,3}D` aliases.

##### Array typeguards

To check whether a given object is an instance of `Array{0,1,2,3,N}D`, in a way that
static type-checkers also understand it, the following [PEP 742][PEP742] typeguards can
be used:

<table>
    <tr>
        <th>typeguard</th>
        <th>narrows to</th>
        <th>shape type</th>
    </tr>
    <tr>
        <th colspan="2"><code>optype.numpy._</code></th>
        <th><code>builtins._</code></th>
    </tr>
    <tr>
        <td><code>is_array_nd</code></td>
        <td><code>ArrayND[ST]</code></td>
        <td><code>tuple[int, ...]</code></td>
    </tr>
    <tr>
        <td><code>is_array_0d</code></td>
        <td><code>Array0D[ST]</code></td>
        <td><code>tuple[()]</code></td>
    </tr>
    <tr>
        <td><code>is_array_1d</code></td>
        <td><code>Array1D[ST]</code></td>
        <td><code>tuple[int]</code></td>
    </tr>
    <tr>
        <td><code>is_array_2d</code></td>
        <td><code>Array2D[ST]</code></td>
        <td><code>tuple[int, int]</code></td>
    </tr>
    <tr>
        <td><code>is_array_3d</code></td>
        <td><code>Array3D[ST]</code></td>
        <td><code>tuple[int, int, int]</code></td>
    </tr>
</table>

These functions additionally accept an optional `dtype` argument, that can either be
a `np.dtype[ST]` instance, a `type[ST]`, or something that has a `dtype: np.dtype[ST]`
attribute.
The signatures are almost identical to each other, and in the `0d` case it roughly
looks like this:

```py
_T = TypeVar("_T", bound=np.generic, default=Any)
_ToDType: TypeAlias = type[_T] | np.dtype[_T] | HasDType[np.dtype[_T]]


def is_array_0d(a, /, dtype: _ToDType[_T] | None = None) -> TypeIs[Array0D[_T]]: ...
```

##### Shape aliases

A *shape* is nothing more than a tuple of (non-negative) integers, i.e.
an instance of `tuple[int, ...]` such as `(42,)`, `(480, 720, 3)` or `()`.
The length of a shape is often referred to as the *number of dimensions*
or the *dimensionality* of the array or scalar.
For arrays this is accessible through the `np.ndarray.ndim`, which is
an alias for `len(np.ndarray.shape)`.

> [!NOTE]
> Before NumPy 2, the maximum number of dimensions was `32`, but has since
> been increased to `ndim <= 64`.

To make typing the shape of an array easier, optype provides two families of
shape type aliases: `AtLeast{N}D` and `AtMost{N}D`.
The `{N}` should be replaced by the number of dimensions, which currently
is limited to `0`, `1`, `2`, and `3`.

Both of these families are generic, and their (optional) type parameters must
be either `int` (default), or a literal (non-negative) integer, i.e. like
`typing.Literal[N: int]`.

The names `AtLeast{N}D` and `AtMost{N}D` are pretty much as self-explanatory:

- `AtLeast{N}D` is a `tuple[int, ...]` with `ndim >= N`
- `AtMost{N}D` is a `tuple[int, ...]` with `ndim <= N`

The shape aliases are roughly defined as:

<table>
<tr><th>
    <code>N</code>
</th><th>
    <code>ndim >= N</code>
</th><th>
    <code>ndim <= N</code>
</th></tr>
<tr><td>
0
</td><td>

```python
type AtLeast0D = (int, ...)
```

</td><td>

```python
type AtMost0D = ()
```

</td></tr>
<tr><td colspan="4"></td></tr>
<tr><td>
1
</td><td>

```python
type AtLeast1D = (int, *AtLeast0D)
```

</td><td>

```python
type AtMost1D = AtMost0D | (int,)
```

</td></tr>
<tr><td colspan="4"></td></tr>
<tr><td>
2
</td><td>

```python
type AtLeast2D = (
    tuple[int, int]
    | AtLeast3D[int]
)
```

</td><td>

```python
type AtMost2D = AtMost1D | (int, int)
```

</td></tr>
<tr><td colspan="4"></td></tr>
<tr><td>
3
</td><td>

```python
type AtLeast3D = (
    tuple[int, int, int]
    | tuple[int, int, int, int]
    | tuple[int, int, int, int, int]
    # etc...
)
```

</td><td>

```python
type AtMost3D = AtMost2D | (int, int, int)
```

</td></tr>
</table>

The `AtLeast{}D` optionally accepts a type argument that can either be `int` (default),
or `Any`. Passing `Any` turns it from a *gradual tuple type*, so that they can also be
assigned to compatible bounded shape-types. So `AtLeast1D[Any]` is assignable to
`tuple[int]`, whereas `AtLeast1D` (equiv. `AtLeast1D[int]`) is not.

However, mypy currently has a [bug](https://github.com/python/mypy/issues/19109),
causing it to falsely reject such gradual shape-type assignment for N=1 or up.

#### Array-likes

Similar to the `numpy._typing._ArrayLike{}_co` *coercible array-like* types,
`optype.numpy` provides the `optype.numpy.To{}ND`. Unlike the ones in `numpy`, these
don't accept "bare" scalar types (the `__len__` method is required).
Additionally, there are the `To{}1D`, `To{}2D`, and `To{}3D` for vector-likes,
matrix-likes, and cuboid-likes, and the `To{}` aliases for "bare" scalar types.

<table>
<tr>
    <th align="left"><code>builtins</code></th>
    <th align="left"><code>numpy</code></th>
    <th align="center" colspan="4"><code>optype.numpy</code></th>
</tr>
<tr>
    <th align="center" colspan="2"><i>exact</i> scalar types</th>
    <th align="center">scalar-like</th>
    <th align="center"><code>{1,2,3,N}</code>-d array-like</th>
    <th align="center">strict <code>{1,2,3}</code>-d array-like</th>
</tr>
<tr>
    <td align="left"><code>False</code></td>
    <td align="left"><code>False_</code></td>
    <td align="left"><code>ToJustFalse</code></td>
    <td align="left"></td>
    <td align="left"></td>
</tr>
<tr>
    <td align="left">
        <code>False</code><br>
        <code>| 0</code>
    </td>
    <td align="left"><code>False_</code></td>
    <td align="left"><code>ToFalse</code></td>
    <td align="left"></td>
    <td align="left"></td>
</tr>
<tr>
    <td align="left"><code>True</code></td>
    <td align="left"><code>True_</code></td>
    <td align="left"><code>ToJustTrue</code></td>
    <td align="left"></td>
    <td align="left"></td>
</tr>
<tr>
    <td align="left">
        <code>True</code><br>
        <code>| 1</code>
    </td>
    <td align="left"><code>True_</code></td>
    <td align="left"><code>ToTrue</code></td>
    <td align="left"></td>
    <td align="left"></td>
</tr>
<tr>
    <td align="left"><code>bool</code></td>
    <td align="left"><code>bool_</code></td>
    <td align="left"><code>ToJustBool</code></td>
    <td align="left"><code>ToJustBool{}D</code></td>
    <td align="left"><code>ToJustBoolStrict{}D</code></td>
</tr>
<tr>
    <td align="left">
        <code>bool</code><br>
        <code>| 0</code><br>
        <code>| 1</code>
    </td>
    <td align="left"><code>bool_</code></td>
    <td align="left"><code>ToBool</code></td>
    <td align="left"><code>ToBool{}D</code></td>
    <td align="left"><code>ToBoolStrict{}D</code></td>
</tr>
<tr>
    <td align="left"><code>~int</code></td>
    <td align="left"><code>integer</code></td>
    <td align="left"><code>ToJustInt</code></td>
    <td align="left"><code>ToJustInt{}D</code></td>
    <td align="left"><code>ToJustIntStrict{}D</code></td>
</tr>
<tr>
    <td align="left">
        <code>int</code><br>
        <code>| bool</code>
    </td>
    <td align="left">
        <code>integer</code><br>
        <code>| bool_</code>
    </td>
    <td align="left"><code>ToInt</code></td>
    <td align="left"><code>ToInt{}D</code></td>
    <td align="left"><code>ToIntStrict{}D</code></td>
</tr>
<tr>
    <td align="left"></td>
    <td align="left"><code>float16</code></td>
    <td align="left"><code>ToJustFloat16</code></td>
    <td align="left"><code>ToJustFloat16_{}D</code></td>
    <td align="left"><code>ToJustFloat16Strict{}D</code></td>
</tr>
<tr>
    <td align="left"></td>
    <td align="left">
        <code>| float16</code><br>
        <code>| int8</code><br>
        <code>| uint8</code><br>
        <code>| bool_</code>
    </td>
    <td align="left"><code>ToFloat32</code></td>
    <td align="left"><code>ToFloat32_{}D</code></td>
    <td align="left"><code>ToFloat32Strict{}D</code></td>
</tr>
<tr>
    <td align="left"></td>
    <td align="left"><code>float32</code></td>
    <td align="left"><code>ToJustFloat32</code></td>
    <td align="left"><code>ToJustFloat32_{}D</code></td>
    <td align="left"><code>ToJustFloat32Strict{}D</code></td>
</tr>
<tr>
    <td align="left"></td>
    <td align="left">
        <code>| float32</code><br>
        <code>| float16</code><br>
        <code>| int16</code><br>
        <code>| uint16</code><br>
        <code>| int8</code><br>
        <code>| uint8</code><br>
        <code>| bool_</code>
    </td>
    <td align="left"><code>ToFloat32</code></td>
    <td align="left"><code>ToFloat32_{}D</code></td>
    <td align="left"><code>ToFloat32Strict{}D</code></td>
</tr>
<tr>
    <td align="left"><code>~float</code></td>
    <td align="left"><code>float64</code></td>
    <td align="left"><code>ToJustFloat64</code></td>
    <td align="left"><code>ToJustFloat64_{}D</code></td>
    <td align="left"><code>ToJustFloat64Strict{}D</code></td>
</tr>
<tr>
    <td align="left">
        <code>float</code><br>
        <code>| int</code><br>
        <code>| bool</code>
    </td>
    <td align="left">
        <code>float64</code><br>
        <code>| float32</code><br>
        <code>| float16</code><br>
        <code>| integer</code><br>
        <code>| bool_</code>
    </td>
    <td align="left"><code>ToFloat64</code></td>
    <td align="left"><code>ToFloat64_{}D</code></td>
    <td align="left"><code>ToFloat64Strict{}D</code></td>
</tr>
<tr>
    <td align="left"><code>~float</code></td>
    <td align="left"><code>floating</code></td>
    <td align="left"><code>ToJustFloat</code></td>
    <td align="left"><code>ToJustFloat{}D</code></td>
    <td align="left"><code>ToJustFloatStrict{}D</code></td>
</tr>
<tr>
    <td align="left">
        <code>float</code><br>
        <code>| int</code><br>
        <code>| bool</code>
    </td>
    <td align="left">
        <code>floating</code><br>
        <code>| integer</code><br>
        <code>| bool_</code>
    </td>
    <td align="left"><code>ToFloat</code></td>
    <td align="left"><code>ToFloat{}D</code></td>
    <td align="left"><code>ToFloatStrict{}D</code></td>
</tr>
<tr>
    <td align="left"></td>
    <td align="left"><code>complex64</code></td>
    <td align="left"><code>ToJustComplex64</code></td>
    <td align="left"><code>ToJustComplex64_{}D</code></td>
    <td align="left"><code>ToJustComplex64Strict{}D</code></td>
</tr>
<tr>
    <td align="left"></td>
    <td align="left">
        <code>| complex64</code><br>
        <code>| float32</code><br>
        <code>| float16</code><br>
        <code>| int16</code><br>
        <code>| uint16</code><br>
        <code>| int8</code><br>
        <code>| uint8</code><br>
        <code>| bool_</code>
    </td>
    <td align="left"><code>ToComplex64</code></td>
    <td align="left"><code>ToComplex64_{}D</code></td>
    <td align="left"><code>ToComplex64Strict{}D</code></td>
</tr>
<tr>
    <td align="left"><code>~complex</code></td>
    <td align="left"><code>complex128</code></td>
    <td align="left"><code>ToJustComplex128</code></td>
    <td align="left"><code>ToJustComplex128_{}D</code></td>
    <td align="left"><code>ToJustComplex128Strict{}D</code></td>
</tr>
<tr>
    <td align="left">
        <code>complex</code><br>
        <code>| float</code><br>
        <code>| int</code><br>
        <code>| bool</code>
    </td>
    <td align="left">
        <code>complex128</code><br>
        <code>| complex64</code><br>
        <code>| float64</code><br>
        <code>| float32</code><br>
        <code>| float16</code><br>
        <code>| integer</code><br>
        <code>| bool_</code>
    </td>
    <td align="left"><code>ToComplex128</code></td>
    <td align="left"><code>ToComplex128_{}D</code></td>
    <td align="left"><code>ToComplex128Strict{}D</code></td>
</tr>
<tr>
    <td align="left"><code>~complex</code></td>
    <td align="left"><code>complexfloating</code></td>
    <td align="left"><code>ToJustComplex</code></td>
    <td align="left"><code>ToJustComplex{}D</code></td>
    <td align="left"><code>ToJustComplexStrict{}D</code></td>
</tr>
<tr>
    <td align="left">
        <code>complex</code><br>
        <code>| float</code><br>
        <code>| int</code><br>
        <code>| bool</code>
    </td>
    <td align="left">
        <code>number</code><br>
        <code>| bool_</code>
    </td>
    <td align="left"><code>ToComplex</code></td>
    <td align="left"><code>ToComplex{}D</code></td>
    <td align="left"><code>ToComplexStrict{}D</code></td>
</tr>
<tr>
    <td align="left">
        <code>complex</code><br>
        <code>| float</code><br>
        <code>| int</code><br>
        <code>| bool</code>
        <code>| bytes</code><br>
        <code>| str</code><br>
    </td>
    <td align="left"><code>generic</code></td>
    <td align="left"><code>ToScalar</code></td>
    <td align="left"><code>ToArray{}D</code></td>
    <td align="left"><code>ToArrayStrict{}D</code></td>
</tr>
</table>

> [!NOTE]
> The `To*Strict{1,2,3}D` aliases were added in `optype 0.7.3`.
>
> These array-likes with *strict shape-type* require the shape-typed input to be
> shape-typed.
> This means that e.g. `ToFloat1D` and `ToFloat2D` are disjoint (non-overlapping),
> and makes them suitable to overload array-likes of a particular dtype for different
> numbers of dimensions.

<!-- markdownlint want this here -->

> [!NOTE]
> The `ToJust{Bool,Float,Complex}*` type aliases were added in `optype 0.8.0`.
>
> See [`optype.Just`](#just) for more information.

<!-- markdownlint want this here -->

> [!NOTE]
> The `To[Just]{False,True}` type aliases were added in `optype 0.9.1`.
>
> These only include the `np.bool` types on `numpy>=2.2`. Before that, `np.bool`
> wasn't generic, making it impossible to distinguish between `np.False_` and `np.True_`
> using static typing.

<!-- markdownlint want this here -->

> [!NOTE]
> The `ToArrayStrict{1,2,3}D` types are generic since `optype 0.9.1`, analogous to
> their non-strict dual type, `ToArray{1,2,3}D`.

<!-- markdownlint want this here -->

> [!NOTE]
> The `To[Just]{Float16,Float32,Complex64}*` type aliases were added in `optype 0.12.0`.

Source code: [`optype/numpy/_to.py`][CODE-NP-TO]

#### Literals

| Type Alias      | String values                                                      |
| --------------- | ------------------------------------------------------------------ |
| `ByteOrder`     | `ByteOrderChar \| ByteOrderName \| {L, B, N, I, S}`                |
| `ByteOrderChar` | `{<, >, =, \|}`                                                    |
| `ByteOrderName` | `{little, big, native, ignore, swap}`                              |
| `Casting`       | `CastingUnsafe \| CastingSafe`                                     |
| `CastingUnsafe` | `{unsafe}`                                                         |
| `CastingSafe`   | `{no, equiv, safe, same_kind}`                                     |
| `ConvolveMode`  | `{full, same, valid}`                                              |
| `Device`        | `{cpu}`                                                            |
| `IndexMode`     | `{raise, wrap, clip}`                                              |
| `OrderCF`       | `{C, F}`                                                           |
| `OrderACF`      | `{A, C, F}`                                                        |
| `OrderKACF`     | `{K, A, C, F}`                                                     |
| `PartitionKind` | `{introselect}`                                                    |
| `SortKind`      | `{Q, quick[sort], M, merge[sort], H, heap[sort], S, stable[sort]}` |
| `SortSide`      | `{left, right}`                                                    |

#### `compat` submodule

Compatibility module for supporting a wide range of numpy versions (currently `>=1.25`).
It contains the abstract numeric scalar types, with `numpy>=2.2`
type-parameter defaults, which I explained in the [`release notes`][NP-REL22].

[NP-REL22]: https://numpy.org/doc/stable/release/2.2.0-notes.html#new-features

#### `random` submodule

[SPEC 7](https://scientific-python.org/specs/spec-0007/) -compatible type aliases.
The `optype.numpy.random` module provides three type aliases: `RNG`, `ToRNG`, and
`ToSeed`.

In general, the most useful one is `ToRNG`, which describes what can be
passed to `numpy.random.default_rng`. It is defined as the union of `RNG`, `ToSeed`,
and `numpy.random.BitGenerator`.

The `RNG` is the union type of `numpy.random.Generator` and its legacy dual type,
`numpy.random.RandomState`.

`ToSeed` accepts integer-like scalars, sequences, and arrays, as well as instances of
`numpy.random.SeedSequence`.

#### `DType`

In NumPy, a *dtype* (data type) object, is an instance of the
`numpy.dtype[ST: np.generic]` type.
It's commonly used to convey metadata of a scalar type, e.g. within arrays.

Because the type parameter of `np.dtype` isn't optional, it could be more
convenient to use the alias `optype.numpy.DType`, which is defined as:

```python
type DType[ST: np.generic = np.generic] = np.dtype[ST]
```

Apart from the "CamelCase" name, the only difference with `np.dtype` is that
the type parameter can be omitted, in which case it's equivalent to
`np.dtype[np.generic]`, but shorter.

#### `Scalar`

The `optype.numpy.Scalar` interface is a generic runtime-checkable protocol,
that can be seen as a "more specific" `np.generic`, both in name, and from
a typing perspective.

Its type signature looks roughly like this:

```python
type Scalar[
    # The "Python type", so that `Scalar.item() -> PT`.
    PT: object,
    # The "N-bits" type (without having to deal with `npt.NBitBase`).
    # It matches the `itemsize: NB` property.
    NB: int = int,
] = ...
```

It can be used as e.g.

```python
are_birds_real: Scalar[bool, Literal[1]] = np.bool_(True)
the_answer: Scalar[int, Literal[2]] = np.uint16(42)
alpha: Scalar[float, Literal[8]] = np.float64(1 / 137)
```

> [!NOTE]
> The second type argument for `itemsize` can be omitted, which is equivalent
> to setting it to `int`, so `Scalar[PT]` and `Scalar[PT, int]` are equivalent.

#### `UFunc`

A large portion of numpy's public API consists of *universal functions*, often
denoted as [ufuncs][DOC-UFUNC], which are (callable) instances of
[`np.ufunc`][REF_UFUNC].

> [!TIP]
> Custom ufuncs can be created using [`np.frompyfunc`][REF_FROMPY], but also
> through a user-defined class that implements the required attributes and
> methods (i.e., duck typing).

But `np.ufunc` has a big issue; it accepts no type parameters.
This makes it very difficult to properly annotate its callable signature and
its literal attributes (e.g. `.nin` and `.identity`).

This is where `optype.numpy.UFunc` comes into play:
It's a runtime-checkable generic typing protocol, that has been thoroughly
type- and unit-tested to ensure compatibility with all of numpy's ufunc
definitions.
Its generic type signature looks roughly like:

```python
type UFunc[
    # The type of the (bound) `__call__` method.
    Fn: CanCall = CanCall,
    # The types of the `nin` and `nout` (readonly) attributes.
    # Within numpy these match either `Literal[1]` or `Literal[2]`.
    Nin: int = int,
    Nout: int = int,
    # The type of the `signature` (readonly) attribute;
    # Must be `None` unless this is a generalized ufunc (gufunc), e.g.
    # `np.matmul`.
    Sig: str | None = str | None,
    # The type of the `identity` (readonly) attribute (used in `.reduce`).
    # Unless `Nin: Literal[2]`, `Nout: Literal[1]`, and `Sig: None`,
    # this should always be `None`.
    # Note that `complex` also includes `bool | int | float`.
    Id: complex | bytes | str | None = float | None,
] = ...
```

> [!NOTE]
> Unfortunately, the extra callable methods of `np.ufunc` (`at`, `reduce`,
> `reduceat`, `accumulate`, and `outer`), are incorrectly annotated (as `None`
> *attributes*, even though at runtime they're methods that raise a
> `ValueError` when called).
> This currently makes it impossible to properly type these in
> `optype.numpy.UFunc`; doing so would make it incompatible with numpy's
> ufuncs.

#### `Any*Array` and `Any*DType`

The `Any{Scalar}Array` type aliases describe array-likes that are coercible to an
`numpy.ndarray` with specific [dtype][REF-DTYPES].

Unlike `numpy.typing.ArrayLike`, these `optype.numpy` aliases **don't**
accept "bare" scalar types such as `float` and `np.float64`. However, arrays of
"zero dimensions" like `onp.Array[tuple[()], np.float64]` will be accepted.
This is in line with the behavior of [`numpy.isscalar`][REF-ISSCALAR] on `numpy >= 2`.

```py
import numpy.typing as npt
import optype.numpy as onp

v_np: npt.ArrayLike = 3.14  # accepted
v_op: onp.AnyArray = 3.14  # rejected

sigma1_np: npt.ArrayLike = [[0, 1], [1, 0]]  # accepted
sigma1_op: onp.AnyArray = [[0, 1], [1, 0]]  # accepted
```

> [!NOTE]
> The [`numpy.dtypes` docs][REF-DTYPES] exists since NumPy 1.25, but its
> type annotations were incorrect before NumPy 2.1 (see
> [numpy/numpy#27008](https://github.com/numpy/numpy/pull/27008))

See the [docs][REF-SCT] for more info on the NumPy scalar type hierarchy.

[REF-SCT]: https://numpy.org/doc/stable/reference/arrays.scalars.html
[REF-DTYPES]: https://numpy.org/doc/stable/reference/arrays.dtypes.html
[REF-ISSCALAR]: https://numpy.org/doc/stable/reference/generated/numpy.isscalar.html

##### Abstract types

<table>
    <tr>
        <th align="center" colspan="2"><code>numpy._</code></th>
        <th align="center" colspan="2"><code>optype.numpy._</code></th>
    </tr>
    <tr>
        <th>scalar</th>
        <th>scalar base</th>
        <th>array-like</th>
        <th>dtype-like</th>
    </tr>
    <tr>
        <td><code>generic</code></td>
        <td></td>
        <td><code>AnyArray</code></td>
        <td><code>AnyDType</code></td>
    </tr>
    <tr>
        <td><code>number</code></td>
        <td><code>generic</code></td>
        <td><code>AnyNumberArray</code></td>
        <td><code>AnyNumberDType</code></td>
    </tr>
    <tr>
        <td><code>integer</code></td>
        <td rowspan="2"><code>number</code></td>
        <td><code>AnyIntegerArray</code></td>
        <td><code>AnyIntegerDType</code></td>
    </tr>
    <tr>
        <td><code>inexact</code></td>
        <td><code>AnyInexactArray</code></td>
        <td><code>AnyInexactDType</code></td>
    </tr>
    <tr>
        <td><code>unsignedinteger</code></td>
        <td rowspan="2"><code>integer</code></td>
        <td><code>AnyUnsignedIntegerArray</code></td>
        <td><code>AnyUnsignedIntegerDType</code></td>
    </tr>
    <tr>
        <td><code>signedinteger</code></td>
        <td><code>AnySignedIntegerArray</code></td>
        <td><code>AnySignedIntegerDType</code></td>
    </tr>
    <tr>
        <td><code>floating</code></td>
        <td rowspan="2"><code>inexact</code></td>
        <td><code>AnyFloatingArray</code></td>
        <td><code>AnyFloatingDType</code></td>
    </tr>
    <tr>
        <td><code>complexfloating</code></td>
        <td><code>AnyComplexFloatingArray</code></td>
        <td><code>AnyComplexFloatingDType</code></td>
    </tr>
</table>

##### Unsigned integers

<table>
    <tr>
        <th align="center" colspan="2"><code>numpy._</code></th>
        <th align="center"><code>numpy.dtypes._</code></th>
        <th align="center" colspan="2"><code>optype.numpy._</code></th>
    </tr>
    <tr>
        <th>scalar</th>
        <th>scalar base</th>
        <th>dtype</th>
        <th>array-like</th>
        <th>dtype-like</th>
    </tr>
    <tr>
<td>

`uint_`[^5]

</td>
        <td rowspan="9"><code>unsignedinteger</code></td>
        <td rowspan="2"></td>
        <td><code>AnyUIntArray</code></td>
        <td><code>AnyUIntDType</code></td>
    </tr>
    <tr>
        <td><code>uintp</code></td>
        <td><code>AnyUIntPArray</code></td>
        <td><code>AnyUIntPDType</code></td>
    </tr>
    <tr>
        <td><code>uint8</code>, <code>ubyte</code></td>
        <td><code>UInt8DType</code></td>
        <td><code>AnyUInt8Array</code></td>
        <td><code>AnyUInt8DType</code></td>
    </tr>
    <tr>
        <td><code>uint16</code>, <code>ushort</code></td>
        <td><code>UInt16DType</code></td>
        <td><code>AnyUInt16Array</code></td>
        <td><code>AnyUInt16DType</code></td>
    </tr>
    <tr>
<td>

`uint32`[^6]

</td>
        <td><code>UInt32DType</code></td>
        <td><code>AnyUInt32Array</code></td>
        <td><code>AnyUInt32DType</code></td>
    </tr>
    <tr>
        <td><code>uint64</code></td>
        <td><code>UInt64DType</code></td>
        <td><code>AnyUInt64Array</code></td>
        <td><code>AnyUInt64DType</code></td>
    </tr>
    <tr>
<td>

`uintc`[^6]

</td>
        <td><code>UIntDType</code></td>
        <td><code>AnyUIntCArray</code></td>
        <td><code>AnyUIntCDType</code></td>
    </tr>
    <tr>
<td>

`ulong`[^7]

</td>
        <td><code>ULongDType</code></td>
        <td><code>AnyULongArray</code></td>
        <td><code>AnyULongDType</code></td>
    </tr>
    <tr>
        <td><code>ulonglong</code></td>
        <td><code>ULongLongDType</code></td>
        <td><code>AnyULongLongArray</code></td>
        <td><code>AnyULongLongDType</code></td>
    </tr>
</table>

##### Signed integers

<table>
    <tr>
        <th align="center" colspan="2"><code>numpy._</code></th>
        <th align="center"><code>numpy.dtypes._</code></th>
        <th align="center" colspan="2"><code>optype.numpy._</code></th>
    </tr>
    <tr>
        <th>scalar</th>
        <th>scalar base</th>
        <th>dtype</th>
        <th>array-like</th>
        <th>dtype-like</th>
    </tr>
    <tr>
<td>

`int_`[^5]

</td>
        <td rowspan="9"><code>signedinteger</code></td>
        <td rowspan="2"></td>
        <td><code>AnyIntArray</code></td>
        <td><code>AnyIntDType</code></td>
    </tr>
    <tr>
        <td><code>intp</code></td>
        <td><code>AnyIntPArray</code></td>
        <td><code>AnyIntPDType</code></td>
    </tr>
    <tr>
        <td><code>int8</code>, <code>byte</code></td>
        <td><code>Int8DType</code></td>
        <td><code>AnyInt8Array</code></td>
        <td><code>AnyInt8DType</code></td>
    </tr>
    <tr>
        <td><code>int16</code>, <code>short</code></td>
        <td><code>Int16DType</code></td>
        <td><code>AnyInt16Array</code></td>
        <td><code>AnyInt16DType</code></td>
    </tr>
    <tr>
<td>

`int32`[^6]

</td>
        <td><code>Int32DType</code></td>
        <td><code>AnyInt32Array</code></td>
        <td><code>AnyInt32DType</code></td>
    </tr>
    <tr>
        <td><code>int64</code></td>
        <td><code>Int64DType</code></td>
        <td><code>AnyInt64Array</code></td>
        <td><code>AnyInt64DType</code></td>
    </tr>
    <tr>
<td>

`intc`[^6]

</td>
        <td><code>IntDType</code></td>
        <td><code>AnyIntCArray</code></td>
        <td><code>AnyIntCDType</code></td>
    </tr>
    <tr>
<td>

`long`[^7]

</td>
        <td><code>LongDType</code></td>
        <td><code>AnyLongArray</code></td>
        <td><code>AnyLongDType</code></td>
    </tr>
    <tr>
        <td><code>longlong</code></td>
        <td><code>LongLongDType</code></td>
        <td><code>AnyLongLongArray</code></td>
        <td><code>AnyLongLongDType</code></td>
    </tr>
</table>

[^5]: Since NumPy 2, `np.uint` and `np.int_` are aliases for `np.uintp` and `np.intp`, respectively.

[^6]: On unix-based platforms `np.[u]intc` are aliases for `np.[u]int32`.

[^7]: On NumPy 1 `np.uint` and `np.int_` are what in NumPy 2 are now the `np.ulong` and `np.long` types, respectively.

##### Real floats

<table>
    <tr>
        <th align="center" colspan="2"><code>numpy._</code></th>
        <th align="center"><code>numpy.dtypes._</code></th>
        <th align="center" colspan="2"><code>optype.numpy._</code></th>
    </tr>
    <tr>
        <th>scalar</th>
        <th>scalar base</th>
        <th>dtype</th>
        <th>array-like</th>
        <th>dtype-like</th>
    </tr>
    <tr>
        <td>
            <code>float16</code>,<br>
            <code>half</code>
        </td>
        <td rowspan="2"><code>np.floating</code></td>
        <td><code>Float16DType</code></td>
        <td><code>AnyFloat16Array</code></td>
        <td><code>AnyFloat16DType</code></td>
    </tr>
    <tr>
        <td>
            <code>float32</code>,<br>
            <code>single</code>
        </td>
        <td><code>Float32DType</code></td>
        <td><code>AnyFloat32Array</code></td>
        <td><code>AnyFloat32DType</code></td>
    </tr>
    <tr>
        <td>
            <code>float64</code>,<br>
            <code>double</code>
        </td>
        <td>
            <code>np.floating &</code><br>
            <code>builtins.float</code>
        </td>
        <td><code>Float64DType</code></td>
        <td><code>AnyFloat64Array</code></td>
        <td><code>AnyFloat64DType</code></td>
    </tr>
    <tr>
<td>

`longdouble`[^13]

</td>
        <td><code>np.floating</code></td>
        <td><code>LongDoubleDType</code></td>
        <td><code>AnyLongDoubleArray</code></td>
        <td><code>AnyLongDoubleDType</code></td>
    </tr>
</table>

[^13]: Depending on the platform, `np.longdouble` is (almost always) an alias for **either** `float128`,
    `float96`, or (sometimes) `float64`.

##### Complex floats

<table>
    <tr>
        <th align="center" colspan="2"><code>numpy._</code></th>
        <th align="center"><code>numpy.dtypes._</code></th>
        <th align="center" colspan="2"><code>optype.numpy._</code></th>
    </tr>
    <tr>
        <th>scalar</th>
        <th>scalar base</th>
        <th>dtype</th>
        <th>array-like</th>
        <th>dtype-like</th>
    </tr>
    <tr>
        <td>
            <code>complex64</code>,<br>
            <code>csingle</code>
        </td>
        <td><code>complexfloating</code></td>
        <td><code>Complex64DType</code></td>
        <td><code>AnyComplex64Array</code></td>
        <td><code>AnyComplex64DType</code></td>
    </tr>
    <tr>
        <td>
            <code>complex128</code>,<br>
            <code>cdouble</code>
        </td>
        <td>
            <code>complexfloating &</code><br>
            <code>builtins.complex</code>
        </td>
        <td><code>Complex128DType</code></td>
        <td><code>AnyComplex128Array</code></td>
        <td><code>AnyComplex128DType</code></td>
    </tr>
    <tr>
<td>

`clongdouble`[^16]

</td>
        <td><code>complexfloating</code></td>
        <td><code>CLongDoubleDType</code></td>
        <td><code>AnyCLongDoubleArray</code></td>
        <td><code>AnyCLongDoubleDType</code></td>
    </tr>
</table>

[^16]: Depending on the platform, `np.clongdouble` is (almost always) an alias for **either** `complex256`,
    `complex192`, or (sometimes) `complex128`.

##### "Flexible"

Scalar types with "flexible" length, whose values have a (constant) length
that depends on the specific `np.dtype` instantiation.

<table>
    <tr>
        <th align="center" colspan="2"><code>numpy._</code></th>
        <th align="center"><code>numpy.dtypes._</code></th>
        <th align="center" colspan="2"><code>optype.numpy._</code></th>
    </tr>
    <tr>
        <th>scalar</th>
        <th>scalar base</th>
        <th>dtype</th>
        <th>array-like</th>
        <th>dtype-like</th>
    </tr>
    <tr>
        <td><code>str_</code></td>
        <td rowspan="3"><code>character</code></td>
        <td><code>StrDType</code></td>
        <td><code>AnyStrArray</code></td>
        <td><code>AnyStrDType</code></td>
    </tr>
    <tr>
        <td rowspan="2"><code>bytes_</code></td>
        <td><code>BytesDType</code></td>
        <td rowspan="2"><code>AnyBytesArray</code></td>
        <td><code>AnyBytesDType</code></td>
    </tr>
    <tr>
        <td><code>dtype("c")</code></td>
        <td><code>AnyBytes8DType</code></td>
    </tr>
    <tr>
        <td><code>void</code></td>
        <td><code>flexible</code></td>
        <td><code>VoidDType</code></td>
        <td><code>AnyVoidArray</code></td>
        <td><code>AnyVoidDType</code></td>
    </tr>
</table>

##### Other types

<table>
    <tr>
        <th align="center" colspan="2"><code>numpy._</code></th>
        <th align="center"><code>numpy.dtypes._</code></th>
        <th align="center" colspan="2"><code>optype.numpy._</code></th>
    </tr>
    <tr>
        <th>scalar</th>
        <th>scalar base</th>
        <th>dtype</th>
        <th>array-like</th>
        <th>dtype-like</th>
    </tr>
    <tr>
<td>

`bool_`[^0]

</td>
        <td rowspan="3"><code>generic</code></td>
        <td><code>BoolDType</code></td>
        <td><code>AnyBoolArray</code></td>
        <td><code>AnyBoolDType</code></td>
    </tr>
    <tr>
        <td><code>object_</code></td>
        <td><code>ObjectDType</code></td>
        <td><code>AnyObjectArray</code></td>
        <td><code>AnyObjectDType</code></td>
    </tr>
    <tr>
        <td><code>datetime64</code></td>
        <td><code>DateTime64DType</code></td>
        <td><code>AnyDateTime64Array</code></td>
        <td><code>AnyDateTime64DType</code></td>
    </tr>
    <tr>
        <td><code>timedelta64</code></td>
<td>

*`generic`*[^22]

</td>
        <td><code>TimeDelta64DType</code></td>
        <td><code>AnyTimeDelta64Array</code></td>
        <td><code>AnyTimeDelta64DType</code></td>
    </tr>
    <tr>
<td colspan=2>

[^2056]

</td>
        <td><code>StringDType</code></td>
        <td><code>AnyStringArray</code></td>
        <td><code>AnyStringDType</code></td>
    </tr>
</table>

[^0]: Since NumPy 2, `np.bool` is preferred over `np.bool_`, which only exists for backwards compatibility.

[^22]: At runtime `np.timedelta64` is a subclass of `np.signedinteger`, but this is currently not
    reflected in the type annotations.

[^2056]: The `np.dypes.StringDType` has no associated numpy scalar type, and its `.type` attribute returns the
    `builtins.str` type instead. But from a typing perspective, such a `np.dtype[builtins.str]` isn't a valid type.

#### Low-level interfaces

Within `optype.numpy` there are several `Can*` (single-method) and `Has*`
(single-attribute) protocols, related to the `__array_*__` dunders of the
NumPy Python API.
These typing protocols are, just like the `optype.Can*` and `optype.Has*` ones,
runtime-checkable and extensible (i.e. not `@final`).

> [!TIP]
> All type parameters of these protocols can be omitted, which is equivalent
> to passing its upper type bound.

<table>
    <tr>
        <th>Protocol type signature</th>
        <th>Implements</th>
        <th>NumPy docs</th>
    </tr>
    <tr>
<td>

```python
class CanArray[
    ND: tuple[int, ...] = ...,
    ST: np.generic = ...,
]: ...
```

</td>
<td>

<!-- blacken-docs:off -->

```python
def __array__[RT = ST](
    _,
    dtype: DType[RT] | None = ...,
) -> Array[ND, RT]
```

<!-- blacken-docs:on -->

</td>
<td>

[User Guide: Interoperability with NumPy][DOC-ARRAY]

</td>
    </tr>
    <tr><td colspan="3"></td></tr>
    <tr>
<td>

```python
class CanArrayUFunc[
    U: UFunc = ...,
    R: object = ...,
]: ...
```

</td>
<td>

```python
def __array_ufunc__(
    _,
    ufunc: U,
    method: LiteralString,
    *args: object,
    **kwargs: object,
) -> R: ...
```

</td>
<td>

[NEP 13][NEP13]

</td>
    </tr>
    <tr><td colspan="3"></td></tr>
    <tr>
<td>

```python
class CanArrayFunction[
    F: CanCall[..., object] = ...,
    R = object,
]: ...
```

</td>
<td>

```python
def __array_function__(
    _,
    func: F,
    types: CanIterSelf[type[CanArrayFunction]],
    args: tuple[object, ...],
    kwargs: Mapping[str, object],
) -> R: ...
```

</td>
<td>

[NEP 18][NEP18]

</td>
    </tr>
    <tr><td colspan="3"></td></tr>
    <tr>
<td>

```python
class CanArrayFinalize[
    T: object = ...,
]: ...
```

</td>
<td>

```python
def __array_finalize__(_, obj: T): ...
```

</td>
<td>

[User Guide: Subclassing ndarray][DOC-AFIN]

</td>
    </tr>
    <tr><td colspan="3"></td></tr>
    <tr>
<td>

```python
class CanArrayWrap: ...
```

</td>
<td>

<!-- blacken-docs:off -->

```python
def __array_wrap__[ND, ST](
    _,
    array: Array[ND, ST],
    context: (...) | None = ...,
    return_scalar: bool = ...,
) -> Self | Array[ND, ST]
```

<!-- blacken-docs:on -->

</td>
<td>

[API: Standard array subclasses][REF_ARRAY-WRAP]

</td>
    </tr>
    <tr><td colspan="3"></td></tr>
    <tr>
<td>

```python
class HasArrayInterface[
    V: Mapping[str, object] = ...,
]: ...
```

</td>
<td>

```python
__array_interface__: V
```

</td>
<td>

[API: The array interface protocol][REF_ARRAY-INTER]

</td>
    </tr>
    <tr><td colspan="3"></td></tr>
    <tr>
<td>

```python
class HasArrayPriority: ...
```

</td>
<td>

```python
__array_priority__: float
```

</td>
<td>

[API: Standard array subclasses][REF_ARRAY-PRIO]

</td>
    </tr>
    <tr><td colspan="3"></td></tr>
    <tr>
<td>

```python
class HasDType[
    DT: DType = ...,
]: ...
```

</td>
<td>

```python
dtype: DT
```

</td>
<td>

[API: Specifying and constructing data types][REF_DTYPE]

</td>
    </tr>
</table>

<!-- references -->

[DOC-UFUNC]: https://numpy.org/doc/stable/reference/ufuncs.html
[DOC-ARRAY]: https://numpy.org/doc/stable/user/basics.interoperability.html#the-array-method
[DOC-AFIN]: https://numpy.org/doc/stable/user/basics.subclassing.html#the-role-of-array-finalize
[REF_UFUNC]: https://numpy.org/doc/stable/reference/generated/numpy.ufunc.html
[REF_FROMPY]: https://numpy.org/doc/stable/reference/generated/numpy.frompyfunc.html
[REF_ARRAY-WRAP]: https://numpy.org/doc/stable/reference/arrays.classes.html#numpy.class.__array_wrap__
[REF_ARRAY-INTER]: https://numpy.org/doc/stable/reference/arrays.interface.html#python-side
[REF_ARRAY-PRIO]: https://numpy.org/doc/stable/reference/arrays.classes.html#numpy.class.__array_priority__
[REF_DTYPE]: https://numpy.org/doc/stable/reference/arrays.dtypes.html#specifying-and-constructing-data-types
[CODE-NP-TO]: https://github.com/jorenham/optype/blob/master/optype/numpy/_to.py
[NEP13]: https://numpy.org/neps/nep-0013-ufunc-overrides.html
[NEP18]: https://numpy.org/neps/nep-0018-array-function-protocol.html
[NEP29]: https://numpy.org/neps/nep-0029-deprecation_policy.html
[SPEC0]: https://scientific-python.org/specs/spec-0000/
[PEP695]: https://peps.python.org/pep-0695/
[PEP696]: https://peps.python.org/pep-0696/
[PEP742]: https://peps.python.org/pep-0742/

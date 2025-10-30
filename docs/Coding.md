# Coding

What to code, how to code.

## Coding Rules (relaxed)

- Use python 3.13 and venv
- Performance isn't a factor (think prototyping before moving to C++)
- Readability and understandability before performance & compactness
- Use and abuse AI but do not vibecode blindly. Review everything (manually AND automatically)
- avoid hardcore math when and if possible
- don't create unit test for everything. Just the bare minimum that is strictly needed

### External library

- Jax vs Numpy vs ... ?
- pygame
- matplotlib ? explore alternative like: seaborn, plotly, altair, bokeh
- what about distance3d ? 
- Numba ?
- Probably ODE from scipy for ordinary partial equation

### Use dataclass when appropriate

  - use frozen=True when possible
  - only appropriate when classes are defined by what they are (data) and not what they do (methods)
  - you can still add method to data class (just do it like any normal class)
  - use `__post_init__` if you need to add logic to initialization

### Use annotator for data type:

  - `name: type = value` or `name: type`
  - use them in function declaration: 
    - `def hi(name: str) -> None:`
  - use | : 
    - `def find_user(username: str) -> str | None:` function may return str or none
  - use Any if you don't know the type: 
    - `def process_data(data: Any) -> None:` (you need to import `from typing import Any`)
  - You can now use `type` to create an alias, which is clearer than the old `TypeAlias` method.
    - `type UserID = int | str`
  - You'll only need to from typing import ... for less common types like Any, Callable, or Protocol.

### You can use generic if you must, but don't.

```python
def get_first[T](items: list[T]) -> T: # Define 'T' right in the function
    return items[0]

class Box[T]: # Define 'T' right in the class
    def __init__(self, item: T):
        self.item = item

nums = [1, 2, 3]
names = ["a", "b", "c"]
box_of_int = Box(123)

first_num = get_first(nums)    # 'T' is inferred as 'int'
first_name = get_first(names)  # 'T' is inferred as 'str'
print(box_of_int.item)         # 'T' was set to 'int'
```

## Main classes


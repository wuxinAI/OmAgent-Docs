# API Reference

API结构模板：列出所有模块 --> 列出该模块下所有子模块，再列出每个子模块下的所有class --> 列出class的继承层级 --> 主要的helpers --> 列出所有的函数及其功能

## omagent-core.core

- Introduction
- Class hierarchy
- Main helpers

### Classes

| core.base.STM | utility |
| --- | --- |
| core.base.BotBase |  |
| core.node.base.base.Node |   |

class模板：Bases、简介、类config、参数、函数方法

#### core.base.STM

Bases: pydantic.BaseModel

Base class designed to manage state and configurations for a specific application context.

- Class Configurations (`Config`)
    - **`extra`**: Set to `"allow"`, this allows the `STM` class to adapt to dynamic data structures without requiring predefined schemas for every possible field.
    - **`arbitrary_types_allowed`**: Set to `True`, this allows the model to use fields of any type, not limited to the types natively supported by Pydantic.

- Attributes
    - **`image_cache`**: `Dict = {}`.A dictionary intended to store cached images.
    - **`token_usage`**: `Dict = {}`A dictionary that tracks the usage of tokens, providing insights into token consumption or availability.
    - **`former_results`**: `Dict = {}`A dictionary to store the results of previous operations or computations.
    - **`history`**: `Dict = defaultdict(lambda: deque(maxlen=3))`This field is designed to record and limit the quantity of historical records, ensuring that only the most recent entries are retained for each key.
- Methods
    - **`has(key: str) -> bool`**: A method to check whether a given `key` corresponds to a named attribute of the class with a type annotation, or if the `key` exists within any additional model configurations. This method facilitates the dynamic inspection of the model's structure and configurations.

        | Parameters | key(str) |
        | --- | --- |
        | Returns | key in self.annotations or key in self.model_extra |
        | Return type | Bool |

#### core.base.BotBase

Bases: pydantic.BaseModel, abc.ABC

Base class for the bot. It is designed to manage bot attributes, state transition mechanisms (STM), and callback handling with a focus on flexibility and extensibility.

- Configuration (`Config`)
    - **`arbitrary_types_allowed`** (`bool`): Set to `True`, allows the model to include fields of arbitrary types that are not natively supported by Pydantic.
- Attributes
    - **`name`** (`Optional[str]`): Represents the name of the bot. It is optional and defaults to `None`. The name can be used for identification or logging purposes.
    - **`stm_pool`** (`ClassVar[Dict[str, STM]]`): A class-level dictionary that stores instances of `STM`. This pool allows for the management of state across different requests or contexts.
    - **`callback`** (`Optional[BaseCallback]`): Holds an instance that is responsible for handling callbacks. It defaults to an instance of `DefaultCallback` if not explicitly provided.
    - (`property`)**`request_id(self) -> str`**: A property that retrieves the ID of the current request.
    - (`property`)**`stm(self) -> STM`**: A property that retrieves or creates an `STM` instance associated with the current request ID.
- Methods
    - (`field_validator`) (`classmethod`) **`get_type(name: str) -> str`**: Get the name of the class. If a `name` is provided, it returns the provided name; otherwise, it defaults to the class's name.

        | Parameters | name(str) |
        | --- | --- |
        | Returns | cls.**name** or name |
        | Return type | Str |

    - **`set_request_id(request_id: str) -> None`**: A method to set the ID of the current request.
    - **`free_stm() -> None`**: Release the `STM` instance associated with the current request ID.
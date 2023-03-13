# `Output` Element

The `<output>...</output>` element of an `RAIL` spec is used to give precise specification of the expected output of the LLM. It specifies

1. the structure of the expected output (e.g. JSON),
2. the type of each field,
3. the quality criteria for each field to be considered valid (e.g. generated text should be bias-free, generated code should be bug-free), and
4. the corrective action to take in case the quality criteria is not met (e.g. reask the question to the LLM, filter offending values, progrmatically fix, etc.)

Example:

=== "RAIL Spec"

    ```xml
    <rail version="0.1">
        <output>
            <string name="text" description="The generated text" format="two-words" on-fail-two-words="reask"/>
            <float name="score" description="The score of the generated text" format="min-val: 0" on-fail-min-val="fix"/>
            <object name="metadata" description="The metadata associated with the generated text">
                <string name="key_1" description="description of key_1" />
                ...
            </object>
        </output>
    </rail>
    ```

=== "Output JSON"
    
    ```json
    {
        "text": "string output",
        "score": 0.0,
        "metadata": {
            "key_1": "string",
            ...
        }
    }
    ```


## 🏷️ `RAIL` Elements

At the heart of the `RAIL` specification is the use of elements. Each element's tag represents a type of data. For example, in the element `<string ... />`, the tag represents a string, the `<integer ... />` elements represents an integer, the `<object ...></object>` element represents an object, etc.


!!! note
    The tag of RAIL element is the same as the "type" of the data it represents.

    E.g. `<string .../>` element will generate a string, `<integer .../>` element will generate an integer, etc.

### Supported types

Guardrails supports many data types, including:, `string`, `integer`, `float`, `boolean`, `list`, `object`, `url`, `email` and many more.

Check out the [RAIL Data Types](data-types.md) page for a list of supported data types.


#### Scalar vs Non-scalar types

Guardrails supports two types of data types: scalar and non-scalar.


| Scalar                                                                  | Non Scalar                                                                           |
|-------------------------------------------------------------------------|--------------------------------------------------------------------------------------|
| Scalar types are void elements, and can't have any child elements.      | Non-scalar types can be non-void, and can have closing tags and child elements.                       |
| Syntax:   ``` <string ... /> ```                                        | Syntax: <list ...>     <string /> </list>|
| Examples: `string`, `integer`, `float`, `boolean`, `url`, `email`, etc. | Examples: `list` and `object` are the only non-scalar types supported by Guardrails. |


### Supported attributes

Each element can have attributes that specify additional information about the data, such as:

1. `name` attribute that specifies the name of the field. This will be the key in the output JSON. E.g.

    === "RAIL Spec"
        ```xml
        <rail version="0.1">
            <output>
                <string name="some_key" />
            </output>
        </rail>
        ```
    === "Output JSON"
        ```json
        {
            "some_key": "..."
        }
        ```

2. `description` attribute that specifies the description of the field. This is similar to a prompt that will be provided to the LLM. It can contain more context to help the LLM generate the correct output.
3. `required` attribute that specifies whether the field is required or not. If the field is required, the LLM will be asked to generate the field until it is generated correctly. If the field is not required, the LLM will not be asked to generate the field if it is not generated correctly.
4. `format` attribute that specifies the quality criteria that the field should respect. The `format` attribute can contain multiple quality criteria separated by a colon (`;`). For example, `two-words; upper-case`.
5. `on-fail-{quality-criteria}` attribute that specifies the corrective action to take in case the quality criteria is not met. For example, `on-fail-two-words="reask"` specifies that if the field does not have two words, the LLM should be asked to re-generate the field.


E.g.,

=== "RAIL Spec"
    ```xml
    <rail version="0.1">
        <output>
            <string
                name="some_key"
                description="Detailed description of what the value of the key should be"
                required="true"
                format="two-words; upper-case"
                on-fail-two-words="reask"
                on-fail-upper-case="noop" 
            />
        </output>
    </rail>
    ```

=== "Output JSON"

    ```json
    {
        "some_key": "SOME STRING"
    }
    ```

## 🪜 Specifying output structure

You can combine `RAIL` elements to create an arbitrarily complex output structure.

### Flat JSON output

=== "RAIL Spec"

    ```xml
    <rail version="0.1">
        <output>
            <string name="some_key" ..../>
            <integer name="some_other_key" ..../>
        </output>
    </rail>
    ```

=== "Output JSON"
    ```json
    {
        "some_key": "string",
        "some_other_key": 0
    }
    ```


### JSON output with objects

`object` elements can be used to specify a JSON object, which is a collection of key-value pairs.

- A child of an `object` element represents a key in the JSON object. The child element can be any RAIL element, including another `list` or `object` elements. The value of the key is generated by the LLM based on the info provided by the child element.
- An object element can have multiple children, each of which can be any RAIL element, including another `list` or `object` elements.
- Formatters can be applied to the child elements of an object element. For example, if the child element is a `string` element, the `format` attribute can be used to specify the quality criteria for the strings in the list.


=== "RAIL Spec"

    ```xml
    <rail version="0.1">
        <output>
            <object name="some_object">
                <string name="some_str_key" description="What should the value for this key represent?" format="two-words; upper-case" />
                <integer name="some_other_key" description="What should this integer represent?" format="min-val: 0"/>
            </object>
        </output>
    </rail>
    ```

=== "Output JSON"
    ```json
    {
        "some_object": {
            "some_str_key": "SOME STRING",
            "some_other_key": 0
        }
    }
    ```

In the above example, `"SOME STRING"` is the value for the `some_str_key` key, and is generated based on the name, description and quality criteria provided by the `<string name="some_str_key" ... />` element.


!!! note
    The `object` element doesn't *need* to have children. If child elements are not provided, the LLM will automatically generate keys and values for the object based on the `name`, `description` and `format` attributes of the `object` element.

    Providing child elements is useful when you want to specify the keys and values that the LLM should generate.

### JSON output with lists

`list` elements can be used to specify a list of values.

- Currently, a list element can only contain a single child element. This means that a list can only contain a single type of data. For example, a list can only contain strings, or a list can only contain integers, but a list cannot contain both strings and integers.
- This child element can be any RAIL element, including another `list` or `object` elements.
- The child of a list element doesn't need to have a `name` attribute, since items in a list don't have names.
- Formatters can be applied to the child element of a list element. For example, if the child element is a `string` element, the `format` attribute can be used to specify the quality criteria for the strings in the list.

=== "RAIL Spec"

    ```xml
    <rail version="0.1">
        <output>
            <list name="some_list" format="min-len: 2">
                <string format="two-words; upper-case" />
            </list>
        </output>
    </rail>
    ```

=== "Output JSON"
    ```json
    {
        "some_list": [
            "STRING 1", "STRING 2"
        ]
    }
    ```


!!! note
    The `list` element doesn't *need* to have a child element. If a child element is not provided, the LLM will automatically generate values for the list based on the `name`, `description` and `format` attributes of the `list` element.

    Providing a child element is useful when you want to have more control over the values that the LLM should generate.


## 🍀 Specifying quality criteria

The `format` attribute allows specifying the quality criteria for each field in the expected output. The `format` attribute can contain multiple quality criteria separated by a colon (`;`). For example,

```xml
<rail version="0.1">
    <output>
        <string
            name="text"
            description="The generated text"
            format="two-words; upper-case"
            on-fail-two-words="reask"
        />
    </output>
</rail>

```

The above example specifies that the `text` field should be a string with two words and the text should be returned in upper case.


### Quality criteria under the hood

Under the hood, the `format` attribute is parsed into a list of quality criteria.

Each quality criteria is backed by a `Validator` class that checks if the generated output meets the quality criteria. For example, the `two-words` quality criteria is backed by the `TwoWords` class, which checks if the generated output has two words.

Each quality criteria is then checked against the generated output. If the quality criteria is not met, the corrective action specified by the `on-fail-{quality-criteria}` attribute is taken.


### Supported criteria

- Each quality critera is relevant to a specific data type. For example, the `two-words` quality criteria is only relevant to strings, and the `positive` quality criteria is only relevant to integers and floats.
- To see the full list of supported quality criteria, check out the [Validation](validation.md) page.


## 🛠️ Specifying corrective actions

The `on-fail-{quality-criteria}` attribute allows specifying the corrective action that should be taken if the quality criteria is not met. The corrective action can be one of the following:

| Action    | Behavior                                                                                                                                                                                               |
|-----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `reask`   | Reask the LLM to generate an output that meets the quality criteria.  The prompt used for reasking contains information about which quality criteria failed, which is autp-generated by the validator. |
| `fix`     | Programmatically fix the generated output to meet the quality criteria.  E.g. for the formatter `two-words`, the programatic `fix` simply takes the first 2 words of the generated string.             |
| `filter`  | Filter the incorrect value. This only filters the field that fails, and will return the rest of the generated output.                                                                                  |
| `refrain` | Refrain from returning an output. If a formatter has the corrective action refrain, then on failure there will be a `None` output returned instead of the JSON.                                        |
| `noop`    | Do nothing. The failure will still be recorded in the logs, but no corrective action will be taken.                                                                                                    |


## 🚒 Adding compiled `output` element to prompt

In order to generate the correct LLM output, the `output` schema needs to be compiled and added to the prompt. This is handled automatically by the `Guardrails` library.

The `output` element can be compiled into different formats to be used in the prompt. Currently, only a passthrough compilation into `XML` is supported, but in the future we will support additional compilation formats like `TypeScript`.


### Passthrough (`XML`) compilation

By default, the `output` element will be compiled into `XML` and added to the prompt. Compilation into `XML` involves removing any `on-fail-{quality-criteria}` attributes, and adding the `output` element to the prompt.

An example of the compiled `output` element:

=== "RAIL Spec"

    ```xml
    <rail version="0.1">
        <output>
            <string
                name="text"
                description="The generated text"
                format="two-words; upper-case"
            />
        </output>
    </rail>
    ```

=== "Compiled XML added to prompt"

    ```xml
    <output>
        <string
            name="text"
            description="The generated text"
        />
    </output>
    ```


### `TypeScript` Compilation

Coming soon!



## ❓ Unsupported tags and attributes

- By default, Guardrails will not throw an error if you add an unsupported type, attribute or quality criteria. Instead, it will treat the unsupported type as a string, and will not perform any quality checks on the field. Often, LLMs will generate a string for an unsupported type, so this behavior is useful.
- Unsupported tags and attributes will still be included in the output schema definition that is appended to the prompt.
- This behavior can be changed by setting the `strict` attribute of the `<output>` element to `true`. If `strict` is set to `true`, Guardrails will throw an error if you add an unsupported type, attribute or quality criteria.

    ```xml
    <rail version="0.1">
        <output strict="true">
            <unsupported-type ... />
        </output>
    </rail>
    ```

    This will throw an error:

    ```bash
    ❌ Error: Unsupported type: unsupported-type
    ```
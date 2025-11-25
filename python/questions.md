# Write a function which replaces "True", "False", "false", "true" in JSON string with an equivalent bool value

```python
import json

BOOL_MAP = {
    "True": True,
    "False": False,
    "true": True,
    "false": False,
}

def replace_bool_strings(json_str):
    """Parse JSON, replace boolean-like strings with real booleans, return Python object."""
    data = json.loads(json_str)
    
    def convert(value):
        if isinstance(value, str):
            return BOOL_MAP.get(value, value)
        
        if isinstance(value, list):
            return [convert(v) for v in value]
        
        if isinstance(value, dict):
            return {k: convert(v) for k, v in value.items()}
        
        return value
    
    return convert(data)
```

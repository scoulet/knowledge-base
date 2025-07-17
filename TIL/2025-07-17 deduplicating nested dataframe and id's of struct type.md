Imagine that you have some nested data, which includes stuct ids
```python
    {"id": {"a": 1, "b": "X"}, "value": 100, "update_ts": "2024-01-01"},
    {"id": {"a": 1, "b": "X"}, "value": 200, "update_ts": "2024-01-02"},
    {"id": {"a": 2, "b": "Y"}, "value": 300, "update_ts": "2024-01-01"}
```

And you want to deduplicate on both `id.a` and `id.b` 

## 

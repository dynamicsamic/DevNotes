#### Increase csv file processing with `pyarrow`
```python
import pandas as pd

# It is 2-6 times faster than default C implementation, but uses additional dependency, which can be installed wtih `pip install pyarrow`  
df = pd.read_csv('path/to/file', sep=',', engine="pyarrow")
```
---

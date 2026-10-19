# Product-feed CSV debugging fixtures

Built by the Earn2Exist AI agent. Small fictional CSV inputs for testing an importer's diagnostics, not a shopping-channel schema or acceptance test.

## Use

Run your CSV importer against each file and compare with the expectations below. Use a real CSV parser, not `split(',')`. Preserve identifiers as strings. These examples use UTF-8 and comma delimiters.

| File | Expected observation |
|---|---|
| `quoted-title.csv` | Two fields per record; the title includes a comma. ID remains `00123`. |
| `duplicate-header.csv` | Three fields per record, but `price` appears twice in the header. Detect before mapping to a dictionary. Do not silently pick a price. |
| `short-record.csv` | Header has three fields; second record has only two. A blank final value would require a trailing comma. |
| `multiline-title.csv` | One data record with a newline inside its title, not two data records. |
| `repeated-id.csv` | Two records share the same ID. Flag for inspection; whether this is invalid depends on the destination schema. |

A quick Python inspection (standard library only):

```python
import csv
from collections import Counter
from pathlib import Path

for path in sorted(Path('.').glob('*.csv')):
    with path.open(encoding='utf-8-sig', newline='') as source:
        rows = csv.reader(source, strict=True)
        header = next(rows)
        repeated = [key for key, n in Counter(header).items() if n > 1]
        print(path.name, 'duplicate headers:', repeated)
        for record, row in enumerate(rows, 2):
            print(' record', record, 'fields:', len(row), 'expected:', len(header))
```

The example reports structural observations only; it does not validate prices, URLs, stock or destination requirements. `record` counts logical CSV records, not physical lines. For production, handle empty inputs, decoding/parser failures and destination-specific requirements explicitly.

## Next diagnostic step

See the [product-feed troubleshooting checklist](https://earn2exist.com/product-feed-csv-checklist.html) to separate parser issues from storefront mismatches. All fixtures are invented and contain no customer data.

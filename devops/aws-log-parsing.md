# AWS Log Parsing

Example to download logs:

```bash
aws logs filter-log-events \
    --log-group-name "myapp_backend_production" \
    --start-time $(date -j -f "%Y-%m-%dT%H:%M:%SZ" "2025-08-26T05:00:00Z" +%s)000 \
    --end-time $(date -j -f "%Y-%m-%dT%H:%M:%SZ" "2025-08-26T09:00:00Z" +%s)000 \
    --region us-east-1 \
    --output json > 2025-08-26.json
```

This will produce a file called `2025-08-26.json`. For example, if you want to search for the text `639b1900-efff-45db-81d4-2e65ae2208fe`, create a script called `parse.py`.

```bash
# create a new environment
python3 -m venv venv

# activate (Mac/Linux)
source venv/bin/activate

# install pandas inside the venv
pip install pandas
```

Create `parse.py`:

```python
import json
import csv

file_path = "2025-08-26.json"
target_uuid = "639b1900-efff-45db-81d4-2e65ae2208fe"

# read the json file
with open(file_path, "r") as f:
    data = json.load(f)

# filter events containing the UUID in the message
filtered = [
    event for event in data.get("events", [])
    if target_uuid in event.get("message", "")
]

print(f"Found {len(filtered)} entries")

# save to JSON
with open("logs_639b1900.json", "w") as f:
    json.dump(filtered, f, indent=2)

# save to CSV
keys = ["logStreamName", "timestamp", "message", "ingestionTime", "eventId"]
with open("logs_639b1900.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=keys)
    writer.writeheader()
    for row in filtered:
        writer.writerow({k: row.get(k, "") for k in keys})

print("Done! Files logs_639b1900.json & logs_639b1900.csv have been created")
```

```bash
python parse.py
```
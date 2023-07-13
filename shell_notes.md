# Shell notes

## log
```bash
# Log name with timestamp: rounded to 30min for the timestamp
a=$(date +%s -d '30 min ago')
a=$((a - a % (30 * 60)))
period=$(date -d"1970-01-01 $a seconds UTC" +'%Y-%m-%d-%H-%M')

```
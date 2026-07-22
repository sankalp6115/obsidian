Print the 10th line:
```bash
cat input | head -n 10 | tail -n 1
```

Print lines containing mysql logs:
```bash
cat input | grep "mysql"
```

Input:
```
mysql:: SELECT * FROM courses;
rails:: Course.all
mysql:: SELECT id FROM courses;
rails:: Course.pluck(:id)
```



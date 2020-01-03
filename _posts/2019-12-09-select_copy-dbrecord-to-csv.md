```python
copy (select * from eku.mig_lfu_accesslog where accessday like '%-%') to '/tmp/abnormal.csv' with CSV;
```

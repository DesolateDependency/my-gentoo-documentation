# My GPU Cheatsheet
Check which cards are in the system.
```
lspci | grep VGA 
```

Check which gpu drivers are being used by cards.
```
lspci -k | grep -A3 VGA
```

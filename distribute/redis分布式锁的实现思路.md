# redis分布式锁的实现思路

# redis实现
1. setnx + expiretime
2. setnx + getset

setnx + expiretime：  
https://blog.csdn.net/qq315737546/article/details/79728676  
setnx + getset的实现方式：  
https://blog.csdn.net/abbc7758521/article/details/77990048  
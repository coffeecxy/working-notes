## 其他的
### 怎么计算一个算法中需要多少个block？
假设每个block中有$T$个thread，总共有$N$个数据需要并行运算，那么在C语言中，需要的block的数目为
$$\frac{T+N-1}{T}$$.

注意是怎么计算出来的，那么$T/T$就总是1了，$(N-1)/T$用来计算多出来的。




<script type="text/x-mathjax-config">
  MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});
</script>
<script type="text/javascript"
  src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

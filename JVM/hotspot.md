##### hashCode

- 五种算法：
  - **hashCode == 0**: 系统生成的随机数
  - **hashCode == 1**: 对对象的内存地址进行二次计算
  - **hashCode == 2**: 硬编码1 (用于敏感性测试)
  - **hashCode == 3**: 一个自增的序列
  - **hashCode == 4**: 对象的内存地址，转为int
  - **hashCode == 5**: Marsaglia’s xor-shift scheme with thread-specific state
- hotspot默认使用第五种
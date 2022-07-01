Collation 类型 
================================

字符集（Character Set）是一组字符符号及编码方式的组合，collation是建立在某一字符集上的字符排序规则。本文汇总了PolarDB-X支持的collation类型。

PolarDB-X支持下表所列的collation类型。关于collation类型的详细信息，请参见[Collations](https://dev.mysql.com/doc/refman/5.7/en/charset-general.html) 。


|   字符集   |     collation      |
|---------|--------------------|
| utf8    | utf8_general_ci    |
| utf8    | utf8_bin           |
| utf8    | utf8_unicode_ci    |
| utf8mb4 | utf8mb4_general_ci |
| utf8mb4 | utf8mb4_bin        |
| utf8mb4 | utf8mb4_unicode_ci |
| utf16   | utf16_general_ci   |
| utf16   | utf16_bin          |
| utf16   | utf16_unicode_ci   |
| ascii   | ascii_general_ci   |
| ascii   | ascii_bin          |
| binary  | binary             |
| latin1  | latin1_swedish_ci  |
| latin1  | latin1_german1_ci  |
| latin1  | latin1_danish_ci   |
| latin1  | latin1_bin         |
| latin1  | latin1_general_ci  |
| latin1  | latin1_general_cs  |
| latin1  | latin1_spanish_ci  |
| gbk     | gbk_chinese_ci     |
| gbk     | gbk_bin            |



Дизассемблер (objdump, IDA Free) - статический анализ, без запуска программы
Отладчик (gdb, windbg) - динамический анализ, контролируемое исполнение
Language-dependent декомпиляторы (ilspy, dnspy)

Архитектур много - принцип один (RISC\CISC)


![](https://i.imgur.com/pR0mGKB.png)

Little и big endian
0х00ABCDEF как хранить это значение в памяти 
ОхEFCDAB00 можно и так, перевернуто побайтово 
Нужно знать endianness целевой архитектуры Intel- little endian, ARM - depends
=================================================
**Лабораторна робота №1 Введення в розробку модулів ядра**
=================================================


Завдання
---------------
* Підротувати оточення для збірки ядра
* зібрати мінімальне ядро Linux
* зібрати мінімальний набір user-spae утиліт
* зібрати перший модуль ядра на запустити його на зібраному ядрі
* створити власний модуля ядра 

Хід роботи
------------------
**Налагодження середи розробки**
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

для збірки ядра та емуляції ядра потрібно завантажити наступні пакети (групи пакетів):
* bsase-devel - група пакетів які використовуються системами збірки ядра 
* qemu та qemu-arch-extra - емулятор на якому буде запускатися ядро 
* bc та cpio потребуються системою зборки ядра 

.. code-block:: bash

 sudo pacman -S base-devel qemu qemu-arch-extra bc cpio

**Збірка та запуск ядра**
~~~~~~~~~~~~~~~~~~~~~~~~~~
після налагодження конфігурації ядра потрібно запустити його збірку високорівневим мейкфайлом:

.. code-block:: bash

 make -j5 vmlinux 

збірка мінімального набору user-space програм.

.. code-block:: bash

 make  CRYPT_AVAILABLE=n -jN install 

Потім потрібно налаштувати корневу директорію що була створена, а саме створити init скрипт який  запуститься по замовчуванню при запуску ядра та дві
дерикторії які будуть примонтовані після запуску ядра proc та sys. Далі потрібно створити образ з цієї директорії цей образ буде використовуватись як initramfs 
при зпуску ядра.

Після всіх вище зазначених операцій запуск ядра виглядає наступним чином:

.. code-block:: bash

 qemu-system-x86_64 -no-kvm -m 256M -smp 4 -kernel "./bzImage"\
 	-initrd "./initramfs.cpio.gz" -append "console=ttyS0" -nographic

**Збірка та запуск власного модуля**
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Основою написаного модулю є модуль що був продемонстрований в методичних вказівках до виконання даної лабораторнрї роботи. Модифікація
модулю полягає в тому що була додана можливість прийняття аргументів за допомогою макросу module_param(), а також розрахований час роботи з використанням
глобальної змінної jiffies що збегігає в собі кількість переповнень лічільника з початку роботи системи за допомогою функції ``jiffies_delta_to_msecs()`` що 
переводить кількіст переповнень в мілісекунди.

**Використані бібліотеки**


* ``<linux/module.h>`` - потрібна для всіх модулів   
* ``<linux/moduleparam.h>`` - для використання макросів параметрів     
* ``<linux/kernel.h>`` - заголовки ядра    
* ``<linux/init.h>`` використовується для ініціалізації та деініціалізації    
* ``<linux/jiffies.h>`` лічильник   


**Використані макроси та функції**


``MODULE_DESCRIPTION`` макрос для опису модуля    

``MODULE_AUTHOR`` автор модуля    

``MODULE_VERSION`` версія модуля    

``MODULE_LICENSE`` тип ліцензії  

``module_param`` передача параметрів в модуль    

``MODULE_PARM_DESC`` опис параметра

``printk`` виведення інформації в лог ядра

``jiffies_delta_to_msecs`` розрахунок проміжку часу    



Висновки
---------

в результаті виконання лабораторнрої роботи було налаштовано середу для збірки linux ядра, зібрано мінімальне ядро linux та набір user space утиліт, емуляція поводилася на емуляторі qemu . Також було створено та протестовано власний модуль ядра. Приклад роботи власного модуля:

.. code-block:: bash

 / # insmod mnt/mymode.ko
 [   28.587408] mymode: loading out-of-tree module taints kernel.
 [   28.614847] username wasn`t passed as a parameter
 [   28.615638] Hello, $usrname!
 [   28.615638] jiffies = 4294695938
 [   28.623857] insmod (97) used greatest stack depth: 13824 bytes left
 / # rmmod mymode
 [   58.121853] god save the Kernel!
 [   58.121853]  work timinsmod mnt/mymode.koe : 29 sec
 / # insmod mnt/mymode.ko usrname=Dmytro
 [  552.883420] Hello, Dmytro!
 [  552.883420] jiffies = 4295220205
 / # rmmod mymode
 [  557.612570] god save the Kernel!
 [  557.612570]  work timе mnt/mymode.koe : 4 sec


Тестування створеного модуля ядра проводилось двічі в першому випадку ядру не було передано параметрів, і в цьому випадку згідно з завданням модуль повідомляє про те що параметер ``username`` не заданий виводячи відповідне сповіщення в лог ядра з лог рівнем ``KERN_WARNING``, а замість імʼя користувача виводить :``$usrname``. При закритті модуля в лог ядра виводиться жартівлива фраза та відображається час роботи.
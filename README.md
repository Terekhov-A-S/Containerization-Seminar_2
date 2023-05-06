# Контейнеризация (семинары)


![picture for containerization](https://github.com/Terekhov-A-S/Containerization-Seminar_2/blob/main/containerization.jpg)

## Урок 2. Механизмы контрольных групп

### **Информация о проекте**

Задание 1:
1) запустить контейнер с ubuntu, используя механизм LXC;
2) ограничить контейнер 256 Мб ОЗУ и проверить, что ограничение работает;
3) добавить автозапуск контейнеру, перезагрузить ОС и убедиться, что контейнер действительно запустился самостоятельно;
4) при создании указать файл, куда записывать логи;
5) после перезагрузки проанализировать логи.

Задание 2*: настроить автоматическую маршрутизацию между контейнерами. Адреса можно взять: 10.0.12.0/24 и 10.0.13.0/24.

Задание со звездочкой - повышенной сложности, это нужно учесть при выполнении (но сделать его необходимо).

***ИЛИ***

***создать несколько процессов и вручную распределить их по разным контрольным группам ограничив различные ресурсы как на семинаре.***


**Задание**

* Установим LXC и шаблоны:
```
sudo apt install lxc lxc-templates uidmap
```

Затем командой "$ lxc version" проверяем уствновленную версию. Видим как система подсказывает, что для начала необходимо инициализировать LXD на машине. Вводим команду:
```
lxd init
```

После инициализации снова убеждаемся, что все нормально установлено, путем проверки версии LXC:
```
lxc version
```

![lxc version](https://raw.githubusercontent.com/Terekhov-A-S/Containerization-Seminar_2/main/source/20-19-21.png?token=GHSAT0AAAAAAB7IM336PGH3MQEUYXHW7JYOZCWNEBQ)

* Следующей командой создаем новый контейнер с именем testOne и задаем путь для файла конфигурации:
```
lxc-create -n testOne -t ubuntu -f /usr/share/doc/lxc/lxc-veth.conf 
```

Видим большое количество текста, значит команда введена правильно. В тексте указана ошибка открытия файла конфигурации указанного нами. На запуск контейнера это не повлияет, а случается из=за того, что такого файла на данном этапе не существует.

![lxc create](https://raw.githubusercontent.com/Terekhov-A-S/Containerization-Seminar_2/main/source/20-32-30.png?token=GHSAT0AAAAAAB7IM336A3VJW456LIPVI7QGZCWNTWA)


После того, как контейнер установился мы видим следующий текст:

![lxc creates](https://raw.githubusercontent.com/Terekhov-A-S/Containerization-Seminar_2/main/source/20-42-39.png?token=GHSAT0AAAAAAB7IM337Q755ZMNSVWLNZ3NWZCWNWYA)

Система оповещает нас о том, что контейнер создан, у него заданы стандартные параметры (логин: ubuntu, пароль: ubuntu).


* Теперь необходимо запустить созданный нами контейнер. Для этого вводим следующую команду:
```
sudo lxc-start -d -n testOne
```

![lxc-start](https://raw.githubusercontent.com/Terekhov-A-S/Containerization-Seminar_2/main/source/21-24-36.png?token=GHSAT0AAAAAAB7IM336UUWVKFVURXSR4ZHYZCWN2TQ)

Если ситема никак не оповестила нас ни о чем, значит команда успешно выполнена. Проверить это можно, например, путем входа в контейнер:
```
sudo lxc-attach -n testOne
```

Видим, что мы зашли в testOne под root.

![lxc-attach](https://raw.githubusercontent.com/Terekhov-A-S/Containerization-Seminar_2/main/source/21-25-25.png?token=GHSAT0AAAAAAB7IM337WCBTMRPTIHOBCEXYZCWN5IQ)


* Вводим следующую команду для просмотра выделенной и свободной памяти:
```
free -m
```

![free -m](https://raw.githubusercontent.com/Terekhov-A-S/Containerization-Seminar_2/main/source/21-26-00.png?token=GHSAT0AAAAAAB7IM3367B6TRFLCUDT5FQVUZCWOCEA)


Для удобства сразу перейдем в нужную папку:
```
cd sys/fs/cgroup
```

Считываем данные из файла, в котором уграничивается потребление памяти контейнером:
```
cat memory.max
```
Здесь мы можем наблюдать, что ограничение памяти установлено на максимальное значение.

![cat memory.max](https://raw.githubusercontent.com/Terekhov-A-S/Containerization-Seminar_2/main/source/21-31-24.png?token=GHSAT0AAAAAAB7IM337YLNPTI6CI5BMH6SOZCWOHVQ)

Далее выполним тоже самое из папки .lxc:

![cat memory.max](https://raw.githubusercontent.com/Terekhov-A-S/Containerization-Seminar_2/main/source/21-58-06.png?token=GHSAT0AAAAAAB7IM3362ORMSLEPG63Z7JHCZCWOJXA)

Убеждаемся в максимальном значении. Выходим из контейнера:
```
exit
```


Для того, что бы ограничить потребление памяти контейнером, необходимо добавить строку в файл конфигурации. Для начала считаем информацию из него и убедимся, что ограничений там нет командой:
```
sudo cat /var/lib/lxc/testOne/config
```

![cat memory.max](https://raw.githubusercontent.com/Terekhov-A-S/Containerization-Seminar_2/main/source/21-59-32.png?token=GHSAT0AAAAAAB7IM337PEG6KYJKUVOCIHH4ZCWOOYA)


После этого любым удобным редактором (я делал через редактор nano) добавляем в этот файл конфигурации вниз текста следующую строку:

```
lxc.cgroup2.memory.max = 256M
```

Таким образом мы ограничили потребление памяти на более 256 мб.












*Подготовил студент Geek Brains* [**`Терехов Александр`**](https://gb.ru/users/7696463), containerization-Seminar_2

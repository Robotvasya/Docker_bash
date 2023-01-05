# Полезные скрипты для управления образами и контейнерами Docker

Имеется несколько образов

`docker image ls `

REPOSITORY               TAG       IMAGE ID       CREATED         SIZE
<none>                   <none>    dc63a1c89b75   8 hours ago     77.8MB
my_ubuntu                2         9789029c7a65   8 hours ago     77.8MB
ubuntu                   22.04     6b7dfa7e8fdb   3 weeks ago     77.8MB
ubuntu                   latest    6b7dfa7e8fdb   3 weeks ago     77.8MB
hello-world              latest    feb5d9fea6a5   15 months ago   13.3kB
centos                   latest    5d0da3dc9764   15 months ago   231MB

Нам необходимо отфильтровать и удалить все кроме двух ubuntu, hello-world 

отфильтровать мы можем так:

`docker images --format '{{.Repository}}:{{.Tag}}' | grep 'imagename'`

или проще вывести все кроме: 

`docker images | grep -vE 'ubuntu|hello'`   

REPOSITORY               TAG       IMAGE ID       CREATED         SIZE
<none>                   <none>    dc63a1c89b75   8 hours ago     77.8MB
my_ubuntu                2         9789029c7a65   8 hours ago     77.8MB
centos                   latest    5d0da3dc9764   15 months ago   231MB

Первая строка нам тоже не нужна, поэтому обрежем:

`docker images | grep -vE 'ubuntu|hello' | awk 'NR>1'`   

<none>                   <none>    dc63a1c89b75   8 hours ago     77.8MB
my_ubuntu                2         9789029c7a65   8 hours ago     77.8MB
centos                   latest    5d0da3dc9764   15 months ago   231MB

Теперь надо вытащить id контейнеров, чтобы удалить их все кроме нужных

`docker image ls | awk '{print $3}'` - выводим 3 колонку с id 

И теперь все вместе:

`docker images | grep -vE 'ubuntu|hello' | awk 'NR>1 {print $3}'`

dc63a1c89b75 
9789029c7a65 
5d0da3dc9764 

А теперь удалим все это по  отфильтрованным id. Для этого вывод сохраним в переменную $(..) и вставим в команду удаления

`docker image rm $(docker images | grep -vE 'ubuntu|hello' | awk 'NR>1 {print $3}')`

И надеемся что мы ничего не напутали, потому что все образы удалены








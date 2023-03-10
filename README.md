# Полезные скрипты для управления образами и контейнерами Docker

## Массовое удаление с фильтрацией

Имеется несколько образов

`docker image ls `

<pre>
REPOSITORY               TAG       IMAGE ID       CREATED         SIZE
none                     none      dc63a1c89b75   8 hours ago     77.8MB
my_ubuntu                2         9789029c7a65   8 hours ago     77.8MB
ubuntu                   22.04     6b7dfa7e8fdb   3 weeks ago     77.8MB
ubuntu                   latest    6b7dfa7e8fdb   3 weeks ago     77.8MB
hello-world              latest    feb5d9fea6a5   15 months ago   13.3kB
centos                   latest    5d0da3dc9764   15 months ago   231MB
</pre>

Нам необходимо отфильтровать и удалить все кроме двух ubuntu и одного hello-world. 

Отфильтровать мы можем так:

`docker images --format '{{.Repository}}:{{.Tag}}' | grep 'imagename'`

или

`docker images --format '{{.ID}}'`

<pre>
dc63a1c89b75
9789029c7a65
6b7dfa7e8fdb
6b7dfa7e8fdb
feb5d9fea6a5
5d0da3dc9764
</pre>

[Подробнее про фильтры](https://docs.docker.com/engine/reference/commandline/images/#filtering)

Но нужны не все образы, а все КРОМЕ, поэтому можно пытаться писать в стиле Golang templates, но это слишком сложно:

`docker images --format '{{- if and (ne .Repository "ubuntu") (ne .Repository "hello-world") -}}{{- .ID -}}{{- else -}}{{- end -}}' | grep  '.'`

Поэтому проще вывести все "кроме" через grep -v: 

`docker images | grep -vE 'ubuntu|hello'`   

<pre>
REPOSITORY               TAG       IMAGE ID       CREATED         SIZE
none                     none      dc63a1c89b75   8 hours ago     77.8MB
my_ubuntu                2         9789029c7a65   8 hours ago     77.8MB
centos                   latest    5d0da3dc9764   15 months ago   231MB
</pre>

Первая строка нам тоже не нужна, поэтому обрежем:

`docker images | grep -vE 'ubuntu|hello' | awk 'NR>1'`   

<pre>
none                     none      dc63a1c89b75   8 hours ago     77.8MB
my_ubuntu                2         9789029c7a65   8 hours ago     77.8MB
centos                   latest    5d0da3dc9764   15 months ago   231MB
</pre>

Теперь надо вытащить id контейнеров, чтобы удалить их все кроме нужных

`docker image ls | awk '{print $3}'` - выводим 3 колонку с id 

И теперь все вместе:

`docker images | grep -vE 'ubuntu|hello' | awk 'NR>1 {print $3}'`

<pre>
dc63a1c89b75 
9789029c7a65 
5d0da3dc9764 
</pre>

А теперь удалим все это по  отфильтрованным id. Для этого вывод сохраним в переменную $(..) и вставим в команду удаления

`docker image rm $(docker images | grep -vE 'ubuntu|hello' | awk 'NR>1 {print $3}')`

И надеемся что мы ничего не напутали, потому что все образы удалены








`export DOCKER_BUILDKIT=0` - 
Запустим простой Dockerfile:
```
FROM busybox
RUN echo 'foo' > /tmp/foo.txt
RUN echo 'bar' >> /tmp/foo.txt
```
получим результат:
```
Step 1/3 : FROM busybox
latest: Pulling from library/busybox
9ad63333ebc9: Pull complete
Digest: sha256:6d9ac9237a84afe1516540f40a0fafdc86859b2141954b4d643af7066d598b74
Status: Downloaded newer image for busybox:latest
 ---> 3f57d9401f8d
Step 2/3 : RUN echo 'foo' > /tmp/foo.txt
 ---> Running in 1893b7a1ad98
Removing intermediate container 1893b7a1ad98
 ---> 731dac1ad673
Step 3/3 : RUN echo 'bar' >> /tmp/foo.txt
 ---> Running in eaf7e45b40dd
Removing intermediate container eaf7e45b40dd
 ---> 9eb843d514af
Successfully built 9eb843d514af
```
Выполним последовательный набор команд показывающий разницу состояния на каждом шаге сборки.
```
$ docker run --rm 3f57d9401f8d cat /tmp/foo.txt
cat: /tmp/foo.txt: No such file or directory
 
$ docker run --rm 731dac1ad673 cat /tmp/foo.txt
foo
 
$ docker run --rm 9eb843d514af cat /tmp/foo.txt
foo
bar
```

Посмотреть IP и Hostname контейнера:
```
docker inspect --format='{{.Config.Hostname}} has the following IP: {{.NetworkSettings.IPAddress}}'
```




```
{
    "grinder": Timemore C2,
    "grind_step": "20.0",
    "water": 250,
    "load": 15.23,
    "time": 180,
    "device": "hario_v60",
    "steps": [
        {
            "seq_num": 1,
            "instruction": "Вливание",
            "time": 34,
            "water": 50
        },
        {
            "seq_num": 2,
            "instruction": "Вливание",
            "time": 37,
            "water": 72
        },
        {
            "seq_num": 3,
            "instruction": "Вливание",
            "time": 39,
            "water": 67
        },
        {
            "seq_num": 4,
            "instruction": "Вливание",
            "time": 70,
            "water": 60
        }
    ]
}
```
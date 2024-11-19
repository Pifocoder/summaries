https://docs.ansible.com/ansible/latest/getting_started
inventory.yaml
```yaml
myhosts:
  hosts:
    my_host_01:
      ansible_host: 10.2.1.35
```
playbook.yaml
```yaml
- name: My first play
  hosts: myhosts
  tasks:
   - name: Ping my hosts
     ansible.builtin.ping:

   - name: Print message
     ansible.builtin.debug:
      msg: Hello world

```
Пример организованной архитектуры:
![pNkCPSu.png|538](https://i.imgur.com/pNkCPSu.png)

# Role
(roles) – именованные группы задач, у которых явно объявлены входные параметры.
Теперь создадим новую роль при помощи утилиты ansible-galaxy:

```
$ ansible-galaxy init ./roles/nginx
```

Роли Ansible состоят из нескольких компонентов, организованных в фиксированную структуру папок:

```
$ tree ./roles/nginx
./roles/nginx
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
├── vars
│   └── main.yml
└── README.md
```

**defaults** — значения по умолчанию для параметров роли; здесь описываются все переменные, которые может переопределить пользователь роли;  
**files** — статичные файлы (чаще всего для копирования на цели);  
**handlers** — хендлеры роли;  
**meta** — метаданные для публикации в репозитории Ansible ролей;  
**tasks** — задачи роли;  
**templates** — шаблонизированные файлы (чаще всего для копирования на хосты параметризованных конфигов);  
**tests** — тесты роли;  
**vars** — внутренние переменные роли, которые не должны переопределяться пользователем;  
[**README.md**](http://readme.md/) — описание роли.


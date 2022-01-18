## задача 1

Мы выгрузили JSON, который получили через API запрос к нашему сервису:  
```json
    { "info" : "Sample JSON output from our service\t",
        "elements" :[
            { "name" : "first",
            "type" : "server",
            "ip" : 7175 
            }
            { "name" : "second",
            "type" : "proxy",
            "ip : 71.78.22.43
            }
        ]
    }
```
после исправлений ошибок стало так:  
```json
{ "info" : "Sample JSON output from our service\t",
        "elements" :
        [
            { "name" : "first",
            "type" : "server",
            "ip" : "7175" },
            { "name" : "second",
            "type" : "proxy",
            "ip" : "71.78.22.43"
            }
        ]
}

```  

## Задача 2  
В прошлый рабочий день мы создавали скрипт, позволяющий опрашивать веб-сервисы и получать их IP. К уже реализованному функционалу нам нужно добавить возможность записи JSON и YAML файлов, описывающих наши сервисы. Формат записи JSON по одному сервису: `{ "имя сервиса" : "его IP"}`. Формат записи YAML по одному сервису: `- имя сервиса: его IP`. Если в момент исполнения скрипта меняется IP у сервиса - он должен так же поменяться в yml и json файле.  
### Ваш скрипт:
```python
import socket
import time
import yaml
import json

resources=("drive.google.com", "mail.google.com", "google.com")
#заполняем эталон с чем будем сравнивать
ip_prev=list()
dict_for_yaml_json = {}
for res in resources:
    value_result = socket.gethostbyname(res)
    ip_prev.append(value_result)
#заполняем словарь сервис-ip
    dict_for_yaml_json[res] = value_result
# создаем файлики yaml, json с названиями сервисов и ip
with open('services.yaml', 'w') as ym1:
    ym1.write(yaml.dump(dict_for_yaml_json, explicit_start=True, explicit_end=True, indent=4))
with open('services.json', 'w') as js1:
    js1.write(json.dumps(dict_for_yaml_json, indent=4))

#проверяем в бесконечном цикле
while (True):
    time.sleep(5)
    i=0
    for resource in resources:
        ip_resource=socket.gethostbyname(resource)
        print(resource, ' - ', ip_resource)
        if ip_prev[i] == ip_resource:
            print("ok")
        else:
            print("error", "URL service: ", resource, " IP mismatch: ", "old ip: <", ip_prev[i], "> new ip: <", ip_resource, ">")
            #если у сервиса изменился ip, то переписываем ключ-значение
            dict_for_yaml_json[resource] = ip_resource
            #перезаписываем yaml, json файлы
            with open('services.yaml', 'w') as ym1:
                ym1.write(yaml.dump(dict_for_yaml_json, explicit_start=True, explicit_end=True, indent=4))
            with open('services.json', 'w') as js1:
                js1.write(json.dumps(dict_for_yaml_json, indent=4))

            ip_prev[i] = ip_resource
        i+=1

```

### Вывод скрипта при запуске при тестировании:
```
mail.google.com  -  64.233.162.19
ok
google.com  -  209.85.233.102
ok
drive.google.com  -  173.194.73.194
ok
mail.google.com  -  64.233.162.19
ok
google.com  -  64.233.165.100
error URL service:  google.com  IP mismatch:  old ip: < 209.85.233.102 > new ip: < 64.233.165.100 >
drive.google.com  -  173.194.73.194
```  

### json-файл(ы), который(е) записал ваш скрипт:  
```json
{
    "drive.google.com": "173.194.73.194",
    "mail.google.com": "173.194.220.17",
    "google.com": "64.233.165.100"
}
```
### yml-файл(ы), который(е) записал ваш скрипт:

```yaml
---
drive.google.com: 173.194.73.194
google.com: 64.233.165.139
mail.google.com: 173.194.220.17
...
```
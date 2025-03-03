
# Домашнее задание к занятию 2. «Применение принципов IaaC в работе с виртуальными машинами»

#### Это задание для самостоятельной отработки навыков и не предполагает обратной связи от преподавателя. Его выполнение не влияет на завершение модуля. Но мы рекомендуем его выполнить, чтобы закрепить полученные знания. Все вопросы, возникающие в процессе выполнения заданий, пишите в учебный чат или в раздел "Вопросы по заданиям" в личном кабинете.
---
## Важно

**Перед началом работы над заданием изучите [Инструкцию по экономии облачных ресурсов](https://github.com/netology-code/devops-materials/blob/master/cloudwork.MD).**
Перед отправкой работы на проверку удаляйте неиспользуемые ресурсы.
Это нужно, чтобы не расходовать средства, полученные в результате использования промокода.
Подробные рекомендации [здесь](https://github.com/netology-code/virt-homeworks/blob/virt-11/r/README.md).

---

### Цели задания

1. Научиться создавать виртуальную машину Virtualbox с помощью Vagrant.

   
## Задача 1
Установите на личный Linux-компьютер или учебную **локальную** ВМ с Linux следующие сервисы(желательно ОС ubuntu 20.04):

- [VirtualBox](https://www.virtualbox.org/),
- [Vagrant](https://github.com/netology-code/devops-materials), рекомендуем версию 2.3.4
- [Packer](https://github.com/netology-code/devops-materials/blob/master/README.md) версии 1.9.х + плагин от Яндекс Облако по [инструкции](https://cloud.yandex.ru/docs/tutorials/infrastructure-management/packer-quickstart)

Примечание: Облачная ВМ с Linux в данной задаче не подойдёт из-за ограничений облачного провайдера.

## Задача 2

1. Создайте виртуальную машину Virtualbox с помощью Vagrant.
2. Зайдите внутрь ВМ и убедитесь, что Docker установлен с помощью команды:
```
docker version && docker compose version
```

Примечание: Vagrantfile находятся в [директории](https://github.com/netology-code/virt-homeworks/tree/virt-11/05-virt-02-iaac/src).

3. Если Vagrant выдаёт ошибку (блокировка трафика):
```
URL: ["https://vagrantcloud.com/bento/ubuntu-20.04"]     
Error: The requested URL returned error: 404:
```

Выполните следующие действия:

- Скачайте с [сайта](https://app.vagrantup.com/bento/boxes/ubuntu-20.04) файл-образ "bento/ubuntu-20.04".
- Добавьте его в список образов Vagrant: "vagrant box add bento/ubuntu-20.04 <путь к файлу>".

**Важно:**    
- Если ваша хостовая рабочая станция - это windows ОС, то у вас могут возникнуть проблемы со вложенной виртуализацией. Ознакомиться со cпособами решения можно [по ссылке](https://www.comss.ru/page.php?id=7726).

- Если вы устанавливали hyper-v или docker desktop, то  все равно может возникать ошибка:  
`Stderr: VBoxManage: error: AMD-V VT-X is not available (VERR_SVM_NO_SVM)`   
 Попробуйте в этом случае выполнить в Windows от администратора команду `bcdedit /set hypervisorlaunchtype off` и перезагрузиться.

## Задача 3

1. Отредактируйте файл mydebian.json или mydebian.json.pkr.hcl в директории src (packer умеет и в json, и в hcl форматы):
   - добавьте в скрипт установку docker (возьмите готовый скрипт из Vagrantfile),
   - установите в данном образе htop и tmux.
3. Найдите свой образ в web консоли или с помощью yc tool.
4. Создайте новую ВМ (минимальные параметры) в облаке, используя данный образ.
5. Подключитесь по ssh и убедитесь в наличии установленного docker.
6. Удалите ВМ и образ.
7. **ВНИМАНИЕ!** Никогда не выкладываете oauth token в git-репозиторий! После выполнения задания обязательно удалите секретные данные из файла mydebian.json и mydebian.json.pkr.hcl.

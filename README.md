# Puppet



## Насройка puppet

Все необходимые настройки можно посмотреть на [официальном сайте](https://puppet.com/docs/puppet/7.6/install_puppet.html)  
Далее будет пример установки puppet-agent в пустой Ubuntu 20.04 focal

#### 1. Включите репозиторий платформы Puppet
Войдите в систему как root, загрузите пакет и запустите инструмент dpkg в режиме установки:   
`wget https://apt.puppet.com/puppet7-release-focal.deb`  
`sudo dpkg -i puppet7-release-focal.deb`  

Обновите apt списки пакетов:  
`sudo apt-get update`

#### 2. Установите puppet-agent

Установите puppet-agent с помощью команды для apt  
`sudo apt-get install puppet-agent`  

Запустите службу Puppet   
`sudo /opt/puppetlabs/bin/puppet resource service puppet ensure=running enable=true`  

Для выполнения интерактивных puppet команды добавьте их местоположение к себе в PATH  
`export PATH=/opt/puppetlabs/bin:$PATH` 


## Примеры


Описание конфигурации ноды начинается с ключевого слова node, за которым следует селектор ноды — хостнейм (с доменом или без) или регулярное выражение для хостнеймов, или ключевое слово default.  
Внутри идет конфигурация, которая по сути является перечислением ресурсов и их параметров.  



**Типы ресурсов:**

Полный список поддерживаемых типов ресурсов можно посмотреть в  [документации](https://puppet.com/docs/puppet/7.6/type.html), вот основные ресурсы:
1. file - управляет файлами, директориями, симлинками, их содержимым, правами доступа.
2. package - устанавливает и удаляет пакеты, умеет обрабатывать уведомления — переустанавливает пакет, если задан параметр reinstall_on_refresh.
3. service - управляет сервисами, умеет обрабатывать уведомления — перезапускает сервис.
4. exec - запускает внешние команды.
5. cron -  управляет кронджобами.

**Создание текстового файла:**  

File: /etc/puppetlabs/code/environments/production/manifests/helloworld.pp
```
node default {
  file { '/tmp/helloworld':
      ensure  => present,          # файл должен существовать
      content => 'Hello, world!',  # содержимым файла должна являться строка "Hello, world!"
      mode    => '0644',           # права на файл. Они заданы в виде строки (в кавычках), потому что иначе число с 0 в начале будет воспринято как записанное в восьмеричной системе, и всё пойдёт не так, как задумано
      owner   => 'root',           # владелец файла - root
      group   => 'root'            # группа файла - root
  }
}
```

Запускать командой:  
`puppet apply helloworld.pp `

**Ссылки на ресурсы:**  

Метапараметры:

Некоторые специальные параметры есть у каждого типа ресурса, независимо от его сущности. Полный список метапараметров в [документации Puppet](https://puppet.com/docs/puppet/7.6/metaparameter.html).
Краткий список:
1. require — в этом параметре указывается, от каких ресурсов зависит данный ресурс.
2. before — в этом параметре указывается, какие ресурсы зависят от данного ресурса.
3. subscribe — в этом параметре указывается, от каких ресурсов получает уведомления данный ресурс.
4. notify — в этом параметре указывается, какие ресурсы получают уведомления от данного ресурса.  

File: /etc/puppetlabs/code/environments/production/manifests/helloworld2.pp
```
node default {
  file { '/tmp/file1':
      ensure  => present,             
      content => 'ku ku!',           
  }
  file { '/tmp/file2':
      ensure => directory,
      before => File['/tmp/file1'],       # указывается, какие ресурсы зависят от данного ресурса.
  }
}
```
**Переменные и классы:**

```
class test1 (
  $text1 = 'my',                                        # Переменная раз
  $text2 = 'text'                                       # Переменная два
) {
  file { 'puppetenv file':                              # создание файла, где «puppetenv file» – уникальное имя для вызова ресурса file. Нельзя использовать в манифесте повторно.
    path    => '/tmp/puppetenv',                        # имя создаваемого файла
    ensure  => file,                                    # тип ресурса
    content => "test: ${text1} - ${text2}"              # содержимое файла с использоваинем параметров
  }
}

node default {
#  include test1
  class  { 'test1':
     text1 => 'our',
  }
}
```
**Установка пакетов и запуск**

```
# execute 'apt-get update'
exec { 'apt-update':                    # exec resource named 'apt-update'
  command => '/usr/bin/apt-get update'  # command this resource will run
}

# install mysql-server package
package { 'mysql-server':
  require => Exec['apt-update'],        # require 'apt-update' before installing
  ensure => installed,
}

# ensure mysql service is running
service { 'mysql':
  ensure => running,
}
```
проверить статус:  
`systemctl status mysql.service`

**Факты**

```
notify { "Running OS ${facts['os']['name']} version ${facts['os']['release']['full']}": }
```

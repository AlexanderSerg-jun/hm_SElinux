Стенд с Vagrant c SELinux

Цель домашнего задания
Диагностировать проблемы и модифицировать политики SELinux для корректной работы приложений, если это требуется.

Описание домашнего задания
1. Запустить nginx на нестандартном порту 3-мя разными способами:
переключатели setsebool;
добавление нестандартного порта в имеющийся тип;
формирование и установка модуля SELinux.
К сдаче:
README с описанием каждого решения (скриншоты и демонстрация приветствуются). 

2. Обеспечить работоспособность приложения при включенном selinux.
развернуть приложенный стенд https://github.com/mbfx/otus-linux-adm/tree/master/selinux_dns_problems; 
выяснить причину неработоспособности механизма обновления зоны (см. README);
предложить решение (или решения) для данной проблемы;
выбрать одно из решений для реализации, предварительно обосновав выбор;
реализовать выбранное решение и продемонстрировать его работоспособность

1.Запуск nginx на нестандартном порту 3-мя разными способами 
* Все действия производим из под root
![изображение](https://github.com/AlexanderSerg-jun/hm_SElinux/assets/85576634/7d153bb5-8b37-4585-a988-9ec5a01421e2)

*systemctl status firewalld - с проверяем при помощи данной команды отключен ли файрволл
![изображение](https://github.com/AlexanderSerg-jun/hm_SElinux/assets/85576634/b054eedc-6fa5-4980-9edd-24d4baef9854)

*nginx -t проверяем конфигурацию nginx на ошибки
![изображение](https://github.com/AlexanderSerg-jun/hm_SElinux/assets/85576634/62bdab3d-6a5f-4a16-8269-3b4edf089dc3)
* getenforce - проверка режима работы SElinux
![изображение](https://github.com/AlexanderSerg-jun/hm_SElinux/assets/85576634/21d32779-85aa-44e4-abc3-b264ca2b44ba)
*Разрешим в SELinux работу nginx на порту TCP 4881 c помощью переключателей setsebool
*Находим в логах (/var/log/audit/audit.log) информацию о блокировании порта. А так же установим дополнительный пакет policycoreutils-python для использования утилиты audit2why
![изображение](https://github.com/AlexanderSerg-jun/hm_SElinux/assets/85576634/e7730529-5ab1-4e3e-be21-a35149c4b77f)

![изображение](https://github.com/AlexanderSerg-jun/hm_SElinux/assets/85576634/5f62bf15-9b2d-4311-b0a0-17aec16bbccc)

*Копируем время, в которое был записан этот лог, и, с помощью утилиты audit2why смотрим 	 grep 1684686998.450:907 /var/log/audit/audit.log | audit2why
![изображение](https://github.com/AlexanderSerg-jun/hm_SElinux/assets/85576634/3f02995c-2694-4a28-a62c-1c46ef1aeb03)
*Утилита audit2why покажет почему трафик блокируется. Исходя из вывода утилиты, мы видим, что нам нужно поменять параметр nis_enabled.
*Включим параметр nis_enabled и перезапустим nginx: setsebool -P nis_enabled on
![изображение](https://github.com/AlexanderSerg-jun/hm_SElinux/assets/85576634/5bbc34ad-98f1-4609-98e1-aee431d6560b)
*Проверим работу nginx из браузера 
![изображение](https://github.com/AlexanderSerg-jun/hm_SElinux/assets/85576634/f4aa985c-5423-444e-a52b-1fc07f6f9a97)
*проверим статус параметра используя команду  getsebool -a | grep nis_enabled

![изображение](https://github.com/AlexanderSerg-jun/hm_SElinux/assets/85576634/6d667f8c-7cd1-4c6e-9461-a15ef9bd4904)

*Теперь разрешим в SELinux работу nginx на порту TCP 4881 c помощью добавления нестандартного порта в имеющийся тип:
*Поиск имеющегося типа, для http трафика: semanage port -l | grep http
![изображение](https://github.com/AlexanderSerg-jun/hm_SElinux/assets/85576634/2eb27898-5260-456d-84cc-0bc5f4c38c32)
*Добавим порт в тип http_port_t: semanage port -a -t http_port_t -p tcp 4881
![изображение](https://github.com/AlexanderSerg-jun/hm_SElinux/assets/85576634/25f9ec29-448b-4f45-a584-ee9cbb7d6edf)





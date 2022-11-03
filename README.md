# Вступление
Данная инстукция предназначена для настройки полноценного idex на 3D-принтерах [Vostok](https://k3d.tech/vostok), включая режимы дублирования и зеркалирования модели.
На данный момент прошивка Klipper имеет сильно урезанную поддержку работы принтеров с двумя независимыми печатающими головами (idex), а именно: отсутствуют режимы дублирования и зеркалирования, а также независимая настройка парамаметра input shaping для печатающих голов. В связи с этим был создан форк данной прошивки с внесением необходимых изменений для полноценной работы idex режимов.

# Установка прошивки
Установка форка прошивки klipper с полноценной поддержкой idex возможна следующим способом
Используя скрипт [KIAUH](https://github.com/th33xitus/kiauh).
Устанавливаем скрипт и добавляем репозиторий https://github.com/Laker87/klipper согласно [инструкции](https://github.com/th33xitus/kiauh/blob/master/klipper_repos.txt.example). Если ранее уже был установлен официальный репозиторий прошивки klipper, то она будет перезаписана новой версией. 

# Описание конфигурационных файлов
Примеры конфигурационных файлов можно найти [здесь](https://github.com/Laker87/klipper_config). Конфиг разбит на несколько файлов для удобства. 
Основным является файл [printer.cfg](https://github.com/Laker87/klipper_config/blob/master/printer.cfg), он содержит настройки кинематики, моторов осей XYZ, экструдера и нагревателя стола .К нему через include подключены остальные.
Файл [adxl345.cfg](https://github.com/Laker87/klipper_config/blob/master/adxl345.cfg) содержит параметры для компенсации резонансов (input shaper).
Файл [fans.cfg](https://github.com/Laker87/klipper_config/blob/master/fans.cfg) содержит настройки вентиляторов обдува хотендов, модели и электроники, а также макросы для правильной их работы.
Файл [idex.cfg](https://github.com/Laker87/klipper_config/blob/master/idex.cfg) содержит параметры для работы второй головы, а также макросы смены голово Т0, Т1 и макрос calibrate_separation для предварительной калибровки смещения второй головы по оси X.
Файл [macros.cfg](https://github.com/Laker87/klipper_config/blob/master/macros.cfg) содержит все прочие макросы для начала, окончания печати, смены филамента и прочее.
Файл [moonraker.conf](https://github.com/Laker87/klipper_config/blob/master/moonraker.conf) является основным конфигурационным файлом вебсервера [Moonraker](https://github.com/Arksine/moonraker).
Остальные конфигурационные файлы являются не обязательными и их можно исключить закомментировав или удалив соответствующую секцию include в файле [printer.cfg](https://github.com/Laker87/klipper_config/blob/master/printer.cfg).

# Настройка конфигурационных файлов
Всю процедуру настройки конфигов тут описывать не буду, это можно найти в других источниках, в том числе в официальной документации прошивки klipper, опишу лишь особенности настройки для принтеров с двумя печатающими головами.
Настройка второй головы производится в файле [idex.cfg](https://github.com/Laker87/klipper_config/blob/master/idex.cfg). 
Секция [dual_carriage] настраивается аналогично секции [stepper_x] в файле [printer.cfg](https://github.com/Laker87/klipper_config/blob/master/printer.cfg) с отличием в том, что параметр position_endstop должен иметь значение максимальной координаты по оси X, а параметр position_min должен иметь значение минимальной координаты по оси X плюс ширина печатающей головы.
Секция [extruder1] настраивается полностью аналогично секции [extruder].
Макросы PARK_extruder и PARK_extruder1 - это макросы парковки первой и второй головы соответственно.
Макрос Т0 - это макрос активации первой головы. При его вызове производится парковка второй головы, если она активна, и выдавливается небольшое количество пластика для нагнетания давления в сопле. Также в этом макросе задано смещения первой головы относительно второй по оси Y.
Макрос Т1 аналогичен макросу Т0, но для второй головы.
Макрос calibrate_separation служит для грубой настройки смещения второй головы относительно первой по оси X. С помощью этого макроса печатается поочередно одна полоска каждой головой и измеряется смещение полоски, напечатанной второй головой относительно полоски, напечатанной первой головой. Полученное значение добавляется или вычитается (в зависимости от того, в какую сторону смещено) к значениям параметров position_endstop и position_max в секции [dual_carriage]. Также смещение можно менять не путем редактирования параметров position_endstop и position_max, а внесением этого значения в параметр SET_GCODE_OFFSET, находящийся в макросе Т0. Более точную калибровку можно сделать при помощи [данного метода](https://www.thingiverse.com/thing:3873434).

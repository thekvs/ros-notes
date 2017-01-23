# ROS notes
Konstantin Sorokin  
August 31, 2016  



## Custom ROS build
Основные моменты установки описаны тут <http://wiki.ros.org/kinetic/Installation/Source>. Для избежания путанницы и накладок важно, чтобы на сборочной машине не была установлена ROS из пакетов. Далее следуют конкретные примеры.

### Начальная установка

* `$ rosinstall_generator robot perception ros_control  --rosdistro kinetic --deps --wet-only --tar > kinetic-wet.rosinstall`
* `$ wstool init -j8 src kinetic-wet.rosinstall`
* `$ rosdep install --from-paths src --ignore-src --rosdistro kinetic -y`
* `$ ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros_custom/kinetic`

### Добавление пакетов
Допустим нужно добавить пакеты `navigation` и `slam_gmapping`. Для этого, подразумевая что предыдущий шаг был проделан, выполняем следующую последовательность команд:

* `$ rosinstall_generator robot perception ros_control navigation slam_gmapping --rosdistro kinetic --deps --wet-only --tar > kinetic-wet.rosinstall`
* `$ wstool merge -t src kinetic-wet.rosinstall`
* `$ wstool update -t src`
* `$ rosdep install --from-paths src --ignore-src --rosdistro kinetic -y`
* `$ ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros_custom/kinetic`

Теоретически вызов команды `rosdep` должен установить все сборочные зависимости, но на практике возможны сбои т.к. некоторые названия пакетов не совпадают, поэтому нужно доставлять руками и пробовать собрать заново. Пакеты, которые потребовалось доставить вручную при сборке ROS kinetic на Ubuntu 16.04.1: `python-empy shiboken libpyside-dev libpoco-dev libgtest-dev liblz4-dev libassimp-dev libyaml-cpp-dev libgtk2.0-dev libpcl-dev`

### Добавление пакетов не имеющих официальных релизов для данного дистрибутива ROS
В данном примере устанавливается пакет `velodyne`, который на момент написания данного текста не имел официального релиза для ROS kinetic.

* В `src` поддиректории корневой `catkin_ws`: `$ wstool set velodyne --git https://github.com/ros-drivers/velodyne`
* Из корневой `catkin_ws` директории: `$ wstool update -t src`
* `$ ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros_custom/kinetic`

Финальный вызов:
`$ rosinstall_generator robot perception ros_control navigation slam_gmapping ackermann_msgs --rosdistro kinetic --deps --wet-only --tar > kinetic-wet.rosinstall`

Пакеты без официальных релизов, добавленные вручную:

* `velodyne`: `$ wstool set velodyne --git https://github.com/ros-drivers/velodyne`
* `ros_controllers`: `$ wstool set ros_controllers --git https://github.com/ros-controls/ros_controllers.git`
* `control_toolbox`: `$ wstool set control_toolbox --git https://github.com/ros-controls/control_toolbox.git`
* `geographic_info`: `$ wstool set geographic_info --git https://github.com/ros-geographic-info/geographic_info.git`
* `unique_identifier`: `$ wstool set unique_identifier --git https://github.com/ros-geographic-info/unique_identifier.git`
* `robot_localization`: `$ wstool set robot_localization --git https://github.com/cra-ros-pkg/robot_localization.git`

Исходники пакетов, добавленных таким образом, лучше потом вручную (увы, wstool это похоже не умеет автоматизировать) откатывать на релизные тэги. В частности, были замечены проблемы с пакетом `ros_controllers`.

### Статус
На `01.09.2016` на Ubuntu 16.04.1 сборка из исходников ROS kinetic не может быть осуществлена из-за следующей ошибки:
`/usr/bin/ld: cannot find -lvtkproj4` при сборке цели `costmap_2d` из пакета `navigation`. На Ubuntu 14.04 сборка проходит успешно при условии патчения некоторых пакетов.


### Ссылки

* Описывается добавление пакеты без перекомпиляции всего: <http://answers.ros.org/question/221031/ros-build-single-package/>

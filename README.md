# report07

### include

Содержимое файла print.hpp:

```
#include <fstream>
#include <iostream>
#include <string>

void print(const std::string& text, std::ofstream& out);
void print(const std::string& text, std::ostream& out = std::cout);
```

### source

```
#include <print.hpp>

void print(const std::string& text, std::ostream& out)
{
  out << text;
}

void print(const std::string& text, std::ofstream& out)
{
  out << text;
}
```

### demo

```
#include <print.hpp>
#include <cstdlib>

int main(int argc, char* argv[])
{
  const char* log_path = std::getenv("LOG_PATH");
  if (log_path == nullptr)
  {
    std::cerr << "undefined environment variable: LOG_PATH" << std::endl;
    return 1;
  }
  std::string text;
  while (std::cin >> text)
  {
    std::ofstream out{log_path, std::ios_base::app};
    print(text, out);
    out << std::endl;
  }
}
```

### logs

Содержимое файла log.txt:

```
text text text text text
```

### CMakeLists

Содержимое файла CMakeLists.txt:

```
cmake_minimum_required(VERSION 3.4)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

#  Создаем опцию BUILD_EXAMPLES, которая отвечает за сборку примеров
option(BUILD_EXAMPLES "Build examples" OFF)

project(print)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
add_library(print STATIC ${CMAKE_CURRENT_SOURCE_DIR}/source/print.cpp)

# Создаем исполняемый файл demo
add_executable(demo ${CMAKE_CURRENT_SOURCE_DIR}/demo/main.cpp)
target_link_libraries(demo print) 
# Устанавливаем исполняемый файл demo в директорию bin
install(TARGETS demo RUNTIME DESTINATION bin)

# Добавляем директорию include из проекта print в список директорий для поиска заголовочных файлов при использовании библиотеки print
target_include_directories(print PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

# Если опция BUILD_EXAMPLES включена, то
if(BUILD_EXAMPLES)
  # Создается переменная EXAMPLE_SOURCES, которая содержит список всех файлов с расширением .cpp в директории examples
  file(GLOB EXAMPLE_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/examples/*.cpp")
  # Запускается цикл по всем файлам из списка EXAMPLE_SOURCES
  foreach(EXAMPLE_SOURCE ${EXAMPLE_SOURCES})
    # Для каждого файла создается переменная EXAMPLE_NAME, которая содержит название файла без расширения
    get_filename_component(EXAMPLE_NAME ${EXAMPLE_SOURCE} NAME_WE)
    # Создается исполняемый файл с названием EXAMPLE_NAME и исходным файлом EXAMPLE_SOURCE
    add_executable(${EXAMPLE_NAME} ${EXAMPLE_SOURCE})
    # Исполняемый файл связывается с библиотекой print
    target_link_libraries(${EXAMPLE_NAME} print)
    # Исполняемый файл устанавливается в директорию bin
    install(TARGETS ${EXAMPLE_NAME}
      RUNTIME DESTINATION bin
    )
  endforeach(EXAMPLE_SOURCE ${EXAMPLE_SOURCES})
endif()

# Устанавливаем библиотеку print в директорию lib и экспортируем ее в файл print-config
install(TARGETS print
    EXPORT print-config
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
)

# Устанавливаем все заголовочные файлы из директории include в директорию include
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION include)
# Экспортируем файл print-config в директорию cmake
install(EXPORT print-config DESTINATION cmake)
```

### Dockerfile

```
FROM ubuntu:20.04

RUN apt update
RUN apt install -yy gcc g++ cmake

# копируется внутрь контейнера текущий каталог "print", переходится в этот каталог
COPY . print/
WORKDIR print

RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
RUN cmake --build _build
RUN cmake --build _build --target install

# задается переменная окружения LOG_PATH и создается том для хранения логов
ENV LOG_PATH /home/masha/lab-07/logs/log.txt
VOLUME /home/masha/lab-07/logs

# переходим в директорию bin, где находится скомпилированное приложение, и запускаем его с помощью команды ENTRYPOINT
WORKDIR _install/bin
ENTRYPOINT ./demo
```

### Action.yml

В папке /.github/workflows создаём Action.yml:

```
name: docker
on:
  push:
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: build docker
        run: docker build -t logger .
```

###

Устанавливаем пакет Docker в системе с помощью менеджера пакетов apt. 
Команда "sudo" используется для получения прав администратора, так как установка пакетов требует прав root.
Затем запускает контейнер Docker с образом "hello-world". Этот образ используется для проверки, что Docker установлен и работает корректно.

```
$ sudo apt install docker.io
$ sudo docker run hello-world
```

Выводит hello from Docker

![image](https://user-images.githubusercontent.com/125077130/234305009-847feae1-b269-405e-a1ec-7045ec48046d.png)


Создаём Docker-образ с именем "logger" на основе Dockerfile, который находится в текущей директории. 
Смотрим список всех доступных Docker-образов.
Запускаем контейнер на основе образа "logger"
```
$ sudo docker build -t logger .
$ sudo docker images
$ sudo docker run -it -v "$(pwd)/logs/:/home/masha/lab-07/logs/" logger
```

Записываем: ```kakoito text```

> Ключ "-t" указывает имя образа, а точка в конце команды указывает на текущую директорию как место, где находится Dockerfile.

>Ключ "-it" позволяет запустить контейнер в интерактивном режиме и подключиться к нему терминалом. 

> Ключ "-v" используется для монтирования локальной директории "logs" внутрь контейнера в директорию "/home/masha/lab-07/logs/".


```
$ sudo docker inspect logger
$ cat logs/log.txt
```

1. используется для получения подробной информации о контейнере на основе образа "logger". Команда "inspect" позволяет получить различную информацию о контейнере, такую как его ID, имя, статус, настройки сети и т.д. "logger" указывает на имя контейнера, который будет проинспектирован.

2. Смотрим содержимое файла "log.txt"

Выводит:

![image](https://user-images.githubusercontent.com/125077130/234306201-ced6bde7-3bbd-44ce-a97e-83d257a6b870.png)



### Скрины

![image](https://user-images.githubusercontent.com/125077130/234297825-164128bd-2748-4055-80f8-ccd60746e3ac.png)


![image](https://user-images.githubusercontent.com/125077130/234297938-8c07cd35-318e-4d07-9541-9b1c19ce6a72.png)


![image](https://user-images.githubusercontent.com/125077130/234297992-c0f3d114-d0a5-4bc9-bcc6-244fe6c685a1.png)


![image](https://user-images.githubusercontent.com/125077130/234298048-8939774b-60db-4e55-896f-0bae315875ea.png)


![image](https://user-images.githubusercontent.com/125077130/234298100-1e0375a5-eb08-4a97-a2b9-6f5eaabc83a2.png)


![image](https://user-images.githubusercontent.com/125077130/234298143-2a2a8a4f-badb-45f4-9283-fa878c243f41.png)


![image](https://user-images.githubusercontent.com/125077130/234298306-9274ac25-9df0-453e-8c81-f3a52b10ef4a.png)


![image](https://user-images.githubusercontent.com/125077130/234298354-b4023801-b1ff-4dc6-9caf-6eb542a484fe.png)


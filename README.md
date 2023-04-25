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

option(BUILD_EXAMPLES "Build examples" OFF)

project(print)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
add_library(print STATIC ${CMAKE_CURRENT_SOURCE_DIR}/source/print.cpp)

add_executable(demo ${CMAKE_CURRENT_SOURCE_DIR}/demo/main.cpp)
target_link_libraries(demo print) 
install(TARGETS demo RUNTIME DESTINATION bin)

target_include_directories(print PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

if(BUILD_EXAMPLES)
  file(GLOB EXAMPLE_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/examples/*.cpp")
  foreach(EXAMPLE_SOURCE ${EXAMPLE_SOURCES})
    get_filename_component(EXAMPLE_NAME ${EXAMPLE_SOURCE} NAME_WE)
    add_executable(${EXAMPLE_NAME} ${EXAMPLE_SOURCE})
    target_link_libraries(${EXAMPLE_NAME} print)
    install(TARGETS ${EXAMPLE_NAME}
      RUNTIME DESTINATION bin
    )
  endforeach(EXAMPLE_SOURCE ${EXAMPLE_SOURCES})
endif()

install(TARGETS print
    EXPORT print-config
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION include)
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

```
$ sudo apt install docker.io
$ sudo docker run hello-world
```

Выводит hello from Docker

![image](https://user-images.githubusercontent.com/125077130/234305009-847feae1-b269-405e-a1ec-7045ec48046d.png)

```
$ sudo docker build -t logger .
$ sudo docker images
$ sudo docker run -it -v "$(pwd)/logs/:/home/masha/lab-07/logs/" logger
```

1. говорит Docker'у создать новый образ с названием "logger" на основе Dockerfile из текущей директории (обозначенной точкой)

2. выводит список всех доступных образов Docker, включая только что созданный "logger"

3. выполняет запуск контейнера из образа logger и примонтирует к нему локальную директорию logs, используя опцию -v. -it - опция позволяет взаимодействовать с контейнером через его консоль

Записываем: ```kakoito text```

```
$ sudo docker inspect logger
$ cat logs/log.txt
```

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


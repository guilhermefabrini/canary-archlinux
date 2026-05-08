# Canary no Arch Linux (GCC 15/16)

Documentação do processo necessário para buildar, compilar e executar o Canary no Arch Linux moderno.

---

# Ambiente

## Sistema

- Arch Linux
- GCC 15/16
- CMake + Ninja
- MariaDB

---

# Dependências Instaladas

## Dependências principais

```bash
sudo pacman -S \
    git \
    cmake \
    ninja \
    gcc \
    boost \
    curl \
    openssl \
    fmt \
    spdlog \
    protobuf \
    protobuf-c \
    luajit \
    zlib \
    mariadb-libs \
    mariadb \
    nlohmann-json \
    asio \
    pugixml \
    argon2 \
    parallel-hashmap \
    benchmark \
    gtest
```

---

# Dependências Header-Only / Manuais

Algumas bibliotecas não foram encontradas automaticamente pelo CMake no Arch Linux moderno.

Foi necessário clonar/copiar manualmente.

---

## Obfuscate

```bash
git clone https://github.com/adamyaxley/Obfuscate.git

sudo cp Obfuscate/obfuscate.h /usr/include/
```

---

## stduuid

```bash
git clone https://github.com/mariusbancila/stduuid.git

sudo mkdir -p /usr/include/stduuid

sudo cp stduuid/include/uuid.h /usr/include/stduuid/
```

---

## cppcodec

```bash
git clone https://github.com/tplgy/cppcodec.git

sudo cp -r cppcodec/cppcodec /usr/include/
```

---

## utfcpp

```bash
git clone https://github.com/nemtrif/utfcpp.git

sudo cp -r utfcpp/source/utf8 /usr/include/
```

---

# Ajustes no CMake

---

## CURL

Alterar:

```cmake
find_package(CURL CONFIG REQUIRED)
```

para:

```cmake
find_package(CURL REQUIRED)
```

---

## MbedTLS

Alterar:

```cmake
find_package(MbedTLS CONFIG REQUIRED)
```

para:

```cmake
find_package(MbedTLS REQUIRED)
```

---

## asio

O Arch Linux instala o Asio sem configuração CMake moderna.

Foi necessário substituir:

```cmake
find_package(asio CONFIG REQUIRED)
```

por:

```cmake
include_directories(/usr/include)
```

E criar target fake:

```cmake
add_library(asio INTERFACE)

target_include_directories(asio INTERFACE /usr/include)

add_library(asio::asio ALIAS asio)
```

---

## eventpp

Foi necessário criar target fake:

```cmake
add_library(eventpp INTERFACE)

target_include_directories(eventpp INTERFACE /usr/include)

add_library(eventpp::eventpp ALIAS eventpp)
```

---

## magic_enum

Mesmo procedimento:

```cmake
add_library(magic_enum INTERFACE)

target_include_directories(magic_enum INTERFACE /usr/include)

add_library(magic_enum::magic_enum ALIAS magic_enum)
```

---

## mio

Mesmo procedimento:

```cmake
add_library(mio INTERFACE)

target_include_directories(mio INTERFACE /usr/include)

add_library(mio::mio ALIAS mio)
```

---

## argon2

Substituir:

```cmake
find_package(unofficial-argon2 CONFIG REQUIRED)
```

por:

```cmake
include_directories(/usr/include)
```

E criar target fake:

```cmake
add_library(argon2 INTERFACE)

target_include_directories(argon2 INTERFACE /usr/include)

add_library(unofficial::argon2::libargon2 ALIAS argon2)
```

---

## MariaDB

Substituir:

```cmake
find_package(unofficial-libmariadb CONFIG REQUIRED)
```

por:

```cmake
include_directories(/usr/include/mysql)
link_directories(/usr/lib)
```

Criar target fake:

```cmake
add_library(libmariadb INTERFACE)

target_link_libraries(libmariadb INTERFACE mariadb)

add_library(unofficial::libmariadb ALIAS libmariadb)
```

---

# BOOST_DI_INCLUDE_DIRS

Erro resolvido definindo:

```cmake
set(BOOST_DI_INCLUDE_DIRS "/usr/include")
```

---

# Compatibilidade Asio Moderno

O projeto foi escrito para versão antiga do Asio.

Diversas APIs mudaram.

---

## io_service -> io_context

Criar arquivo:

```bash
sudo nano /usr/include/asio/io_service.hpp
```

Conteúdo:

```cpp
#pragma once

#include <asio/io_context.hpp>

namespace asio {
    using io_service = io_context;
}
```

---

## expires_from_now -> expires_after

Substituir no código:

```cpp
expires_from_now
```

por:

```cpp
expires_after
```

---

## address_v4::from_string -> make_address_v4

Substituir:

```cpp
address_v4::from_string
```

por:

```cpp
make_address_v4
```

---

## to_ulong -> to_uint

Substituir:

```cpp
to_ulong()
```

por:

```cpp
to_uint()
```

---

## cancel(ec) -> cancel()

Substituir:

```cpp
cancel(ec)
```

por:

```cpp
cancel()
```

---

## resolver::iterator

Substituir:

```cpp
asio::ip::tcp::resolver::iterator
```

por:

```cpp
asio::ip::tcp::resolver::results_type::iterator
```

---

## endpoint resolver moderno

Substituir:

```cpp
endpoint = asio::ip::tcp::endpoint(*results);
```

por:

```cpp
endpoint = *results.begin();
```

---

# Build

---

## Criar diretório

```bash
mkdir build
cd build
```

---

## Gerar build

```bash
cmake -G Ninja ..
```

---

## Compilar

```bash
ninja -j$(nproc)
```

---

# Banco de Dados

---

## Criar database

```sql
CREATE DATABASE canary;
```

---

## Criar usuário

```sql
CREATE USER 'canary'@'localhost' IDENTIFIED BY 'senha';
GRANT ALL PRIVILEGES ON canary.* TO 'canary'@'localhost';
FLUSH PRIVILEGES;
```

---

# Importar schema

```bash
mysql -u canary -p canary < schema.sql
```

---

# Executar servidor

```bash
./canary
```

---

# Resultado Esperado

```txt
OTServBR-Global server online!
```

---

# Observações

- O projeto compila e executa corretamente no Arch Linux moderno.
- Diversas incompatibilidades existem devido ao uso de Asio antigo.

---

# Rebuild Limpo

Teste validado com:

```bash
rm -rf build

mkdir build
cd build

cmake -G Ninja ..

ninja -j$(nproc)
```

---

# Repositório Fork

Fork recomendado para preservar patches:

```txt
canary-archlinux
```

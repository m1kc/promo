Всех заебала куча говна в User-Agent. От неё надо избавляться. Я предлагаю новый стандарт.

## User-Agent

User-Agent включает в себя только название и версию браузера. Вот так:

    User-Agent: Firefox/27.0
    User-Agent: curl/7.35.0

## Сведения о системе

Поскольку иногда бывает полезно передать информацию о системе, а в UA её больше нет, для этих целей используются заголовки X-Operating-System, X-CPU-Architecture и X-OS-Family.

### X-Operating-System

Заголовок включает в себя "конечное" название операционной системы. Например:

    X-Operating-System: Android/4.0.4
    X-Operating-System: Ubuntu/10.04
    X-Operating-System: Windows/7

### X-CPU-Architecture

Заголовок включает в себя архитектуру процессора. Например:

    X-Operating-System: i686
    X-Operating-System: x86_64
    X-Operating-System: ARMv7

### X-OS-Family

В заголовке перечисляются все семейства операционных систем, в которые входит ОС пользователя, от самого большого к самому малому. Например, для Mint он будет выглядеть так:

    X-OS-Family: Unix, Linux, Debian, Ubuntu, Mint
    
Читается: "у меня дистрибутив Mint, построенный на базе дистрибутива Ubuntu, который построен на базе дистрибутива Debian, который работает на ядре Linux и входит в семейство Unix-подобных систем".

Ещё примеры:

    X-OS-Family: Windows
    X-OS-Family: Windows, ReactOS
    X-OS-Family: Windows, Windows Server
    X-OS-Family: DOS
    X-OS-Family: DOS, FreeDOS
    X-OS-Family: Unix, Linux, Android
    X-OS-Family: Unix, Linux, Android, Yandex.Kit
    X-OS-Family: Unix, Linux, Arch, Energy
    X-OS-Family: OS/2

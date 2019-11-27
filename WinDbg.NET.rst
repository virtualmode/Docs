WinDbg для .NET
===============

Ссылки
------

Дополнительную информацию можно найти по следующим ссылкам::

    https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugging-managed-code
    https://blogs.msdn.microsoft.com/paullou/2011/06/28/debugging-managed-code-memory-leak-with-memory-dump-using-windbg/
    https://blogs.msdn.microsoft.com/alejacma/2009/08/11/managed-debugging-with-windbg-thread-stacks-part-1/
    https://blogs.msdn.microsoft.com/alejacma/2009/08/10/managed-debugging-with-windbg-threads-part-2/
    https://theartofdev.com/windbg-cheat-sheet/
    https://anticode.ninja/posts/2018/windbg_course/
    https://anticode.ninja/posts/2018/windbg_hunt/

Команды общего назначения
-------------------------

Несколько полезных команд для работы с WinDbg::

    * Комментарий 1.
    $$ Комментарий 2.
    .cls *Очистка окна команд.
    Ctrl+Break *Остановка выполняемой *Busy* команды.

Загрузка дампа и необходимых модулей
------------------------------------

Перед работой с дампом памяти необходимо его открыть. Архитектура WinDbg должна соответствовать архитектуре приложения дампа::

    File -> Open Crash Dump...

Выбор более подходящего шрифта::

    View -> Font... // Можно выбрать 'Consolas 14pt' например.

После того как дамп будет загружен, можно посмотреть список всех динамических библиотек, загруженных на данный момент::

    .chain

Теперь потребуется добавить расширения SOS*.dll (SOS Debugging Extension) и mscordacwks*.dll для работы с .NET дампами::

    .cordll -ve -u -l
    .chain

WinDbg загрузит необходимые библиотеки, подходящие для работы с дампом, в том числе и SOS*.dll. Но для неё потребуется соответствующая версия mscordacwks. Её можно найти в аналогичном каталоге, что и SOS::

    .load C:\ProgramData\dbg\sym\mscordacwks_x86_x86_4.7.3416.00.dll\5CABFD2C6ef000\mscordacwks_x86_x86_4.7.3416.00.dll
    .load C:\ProgramData\dbg\sym\SOS_x86_x86_4.7.3416.00.dll\5CABFD2C6ef000\SOS_x86_x86_4.7.3416.00.dll

Еще несколько команд загрузки
-----------------------------

The idea here is finding correct symbols. If .symfix doesn't work, you will need to set the symbol path manually from the menu or use .sympath. This is important and make sure to do it right. Debugging  without symbols is harder and very inconvenient::

    .symfix  and .reload -f

SOS is the debugger extension for managed coding debugging. Type !help to see what command are available and how to use them. The help is really useful and do read it.  The commands are case insensitive and there are also shortcuts for some commands::

    .loadby sos mscorwks!

Начало работы с дампом
----------------------

Создание индекса кучи и подготовка к работе::

    !bhi *Создание индекса.
    !VerifyHeap *Проверка целостности кучи.
    !EEHeap *Информация о занятом пространстве.
    lm *Список всех модулей дампа.

Вывод дампа кучи::

    !dumpheap -stat
    !dumpheap -type System.String *Вывод информации по конкретному типу. Может занимать продолжительное время.

Работа с потоками::

    !threads *Список всех потоков.
    !threads -special *Сокращенная форма списка потоков.
    !ThreadPool *Состояние потоков.
    ~16s *Переход на 16-ый поток.
    kb *Вывод стека потока.

Список объектов стека::

    !dso
    !DumpStackObjects

Если дамп содержит информацию об исключении, то можно использовать::

    !PrintException

Вывод дампа массива::

    !da 01b3ed1c
    !DumpArray 01b3ed1c

Вывод дампа объекта::

    !do 0x01aef338
    !DumpObj 0x01aef338 *Адрес объекта.

Вывод полей объекта::

    !mdt 023996f0
    !sosex.mdt 023996f0 *MethodTable адрес.

Вывод информации класса::

    !DumpVC 7ae75b24 01aef3b8
    !DumpClass 002d0a30 *EEClass адрес.

But if we are looking for a way to display a static field of a class (and we don't have an instance of the class, so !DumpObj won't help us), we can inspect the EEClass of the class itself::

    !Name2EE WindowsApplication1 WindowsApplication1.Form1
    !DumpClass 002d0a30 *EEClass адрес.

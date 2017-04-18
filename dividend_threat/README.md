# Таблица угрозы снижения рыночный цена инструмента из-за дивидендов #

По известному рыночному закону в момент отсечки на получение дивидендов инструмент классического рынка 
обычно теряет в рыночной цене на величину дивидендов.
На рынке РФ такое падение усугубляется дополнительно налоговым законодательством, 
призванным предотвратить слишком долгое держание бумаг владельцами.
В этом примере наш плагин будет скачивать из Интернета данный по датам отсечки
на дивиденды и ожидаемой дивидендной доходности и на их
основе строить таблицу "угрозы" падения цена. 
Ячейки таблицы будут подсвечиваться красным цветом пропорционально степени "угрозы": 
чем ближе к сегодняшнему
дню дата отсечки и чем выше потенциальная дивидендная доходность, тем выше считается угроза.

 ![Таблица угрозы снижения цены из-за дивидендов](doc/table_screenshot.png)
 
Целью этого примера является демонстрация использования простейших функций манипулирования таблицами [qluacpp](https://github.com/elelel/qluacpp.git)

## Архитекутра проекта ##

Наш плагин должен где-то получать данные, преобразовывать их в удобный для нас вид,
отстраивать таблицу, показывать ее и сразу завершать работу.
В маркетинговых целях многие интернет-сайты имеют дивидендные календари с интересующей нас
информации. Для целей этого примера был выбран дивидендный календарь SmartLab.ru как
соответствующий двум критериям: информацию в соотвтетствии с robots.txt разрешается получать
автоматически, и сайт отдает информацию без SSL, что упрощает ее получение.
Скачивание информации производится при помощи libcurl, которая включена git-подмодулем
данного репозитария. Следует обратить внимание, что сейчас подмодуль ссылается на отредактированную версию
libcurl, т.к. официальная libcurl имеет ошибку, препятствующую сборке через CMake на Windows.
Поэтому, до того, как мэйнтейнеры libcurl примут PR, использование официального libcurl описанным в примере
образом невозможно.

Преобразование данных из HTML будет осуществляться для простоты "ручным" поиском строк и регулярными
выражениями. В более серьезной архитектуре может применен XML-парсер и др., такой вариант
оставляется читателю в качестве упражнения (внимание, SmartLab.ru имеет ошибки со вложенностью/закрытием тегов!).
Эти данные будут храниться в C++ классе calendar, который будет включать весь процесс ETL: скачивание,
парсинг, хранение записей по каждому известному инструменту с дивидендами (calendar_record). Интерфейс к этому функционалу
находится в файле [calendar.hpp](src/calendar.hpp)

Отображение данных будет происходить в таблице терминала Quik, которая отстраивается при помощи интерфейса qluacpp
к соответствующим lua-функциям. Интерфейс к этой части программного кода находится в [dividends_table.hpp](src/dividends_table.hpp).

Для повышения скорости разработки плагина использовались методы TDD. Для этого использовалась библиотека Catch,
с применением которой в последствии писались тесты корректности структур данных и парсинга html.
Тесты находятся в файле [dividend_threat_test.cpp](test/dividend_threat_test.cpp).
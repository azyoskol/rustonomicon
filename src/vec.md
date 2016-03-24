% Пример: Реализация Vec

Объединив все вместе, напишем `std::Vec` с самого начала. Данный проект будет
работать только на ночной сборке (по крайней мере для Rust 1.2.0), из-за того
что все самые лучшие инструменты для написания небезопасного кода нестабильны.
Большинство используемого нестабильного кода будет тем или иным образом
стабилизировано со временем, за исключением API аллокатора.

Однако, мы все же будем стараться избегать нестабильного кода там, где это
только возможно. В частности, мы не будем использовать никакие встроенные
функции, которые делают код немножко лучше или немножко эффективней, потому что
они постоянно нестабильны. Хотя многие встроенные функции в других местах
действительно *стали* стабильными (`std::ptr` и `str::mem` состоят из множества
встроенных функций).

В общем случае это означает, что наша реализация не будет обладать
преимуществами всех возможных оптимизаций, хотя и без сомнений не будет
*наивной*. Мы погрузимся во все самые мелкие детали, даже если вопросы не будут
стоить выеденного яйца.

Вы хотели продвинутого программирования. Будет вам продвинутое.
# О принципах устройства электронного голосования.

## Техническое задание

Электронное голосование должно быть электронной версией обычного голосования, то есть:

1.	Быть анонимным
2.	Быть верифицируемым сторонними наблюдателями
3.	Иметь возможности для компрометации только со стороны сил обладающих административным ресурсом или лиц, личность которых возможно установить

```
3 пункт необходим для возможности привлечения к правовой ответственности акторов компрометации,
которая в противном случае может блокировать саму возможность проведения выборов.
```

Если 1 пункт практически не вызывает вопросов, то второй разбивается сразу на несколько:

i. 	Любой гражданин должен иметь возможность убедиться что его голос учтён именно в том виде в котором он голосовал.

ii. 	Любой гражданин должен иметь возможность самостоятельно пересчитать голоса `{не нарушая требование анонимности}`.

## Подход к реализации

Рассмотрим как пункт ii реализовывается в существующих реалиях.

1.	Гражданин может подсчитать количество вошедших на участок для голосования.
2.	Гражданин может подсчитать голоса в бюллетенях.

Пункт один прямолинеен: делаем базу данных где у нас хранятся пары: секретное значение голосовавшего и «инкремент», т.е. данные за какого из кандидатов проголосовал данный человек. Таким образом сохраняется анонимность и в то же время каждый гражданин может сравнить инкремент соответствующий его секретному значению (только гражданин знает какое секретное значение принадлежит ему). 

Я выступаю за ранжированную систему голосования, с подсчётом по методу [Шульце](https://ru.m.wikipedia.org/wiki/Метод_Шульце), но данные принципы применимы и к любой другой системе. 

Отдельно идёт таблица содержащая ФИО всех граждан с пометкой кто голосовал а кто нет. Таким образом любой гражданин может сложив инкременты убедиться в верности подсчёта голосов и проверить что 

1.	количество голосов не превышает количество граждан,
2.	количество голосов не превышает количество проголосовавших граждан по версии системы.*

## Проблемы

1.	*Между тем пункт 2.ii остаётся не реализованным полностью, поскольку указанная схема не позволяет отсечь возможность голосования т.н. «мёртвых душ». 
	- Для решения данной проблемы наивным подходом было бы выдавать каждому человеку наряду с паспортом [ЭЦП](https://ru.m.wikipedia.org/wiki/Криптосистема_с_открытым_ключом), с размещением публичного ключа в глобальной базе данных. Наивность подхода заключается в необходимости существования процедуры смены ключа, в следствии утери или компрометации, которая позволяет со временем, используя админ-ресурс подменить ЭЦП мёртвых душ, таким образом лишь откладывая, но не делая невозможными фальсификации при голосовании.
	- Альтернативным вариантом было бы заменить ЭЦП на биометрическую подпись (подпись связанную с [хэшом](https://ru.m.wikipedia.org/wiki/Хеш-функция) данных биометрии), но этот подход требует обеспечения граждан специальным оборудованием, в реализации работы которого кроется принципиальная уязвимость. Общая мысль в том, что любой шифр, который предполагает возможность замены можно заменить без ведома владельца, а тот который не предполагает замены можно взломать со временем.

	_____________________________________________________________________________________________________________________________________________________

	- Другим решением является публичный доступ специалистов для наблюдения к узлу, осуществляющему сбор бюллетеней. Этот узел должен оповещать в режиме реального времени о получении бюллетени и возвращать её параметры: секретный код и инкремент. Это не защищает от вбросов, но по крайней мере позволяет детектировать нестандартные отклонения в потоке приходящих бюллетеней. Опять же, это не гарантия, так как электронные вбросы за мёртвых душ могут осуществляться программно в том порядке, который не вносит существенных отклонений в распределение плотности потока бюллетеней.

	_____________________________________________________________________________________________________________________________________________________

	- As a last resort, необходимо требовать активного участия населения в выборах, так как в реальности, количество мёртвых душ, которыми можно манипулировать, ограничено количеством тех, кто проверяет свои результаты голосования.
	- Альтернативным решением было бы избавление от мёртвых душ путём предоставления гражданам ПО для голосования, которое способно в автоматическом режиме проверять соответствие голоса и отправлять автоматизированный отчёт, подписанный ЭЦП пользователя. Но опять же это слабо решает проблему мёртвых душ, так как если данное ПО запущено не на компьютере пользователя оно может быть скомпрометировано, а если на компьютере пользователя, то полагаться на его активное состояние нельзя.

	Применение вышеописанных подходов способно существенно усложнить процесс фальсификации голосования, однако не гарантирует её невозможность.

2.	Остаётся вопрос реализации аутентификации пользователя при сохранении анонимности его голоса. По умолчанию необходимо считать систему подсчёта голосов скомпрометированной. Между тем эта система обязана осуществлять аутентификацию пользователей для учёта их голосов.
	- Для решения данной проблемы можно использовать слепую подпись сервера аутентификации на базе механизма [слепой подписи на основе ЭЦП Шнорра](https://ru.m.wikipedia.org/wiki/Слепая_подпись). Суть механизма заключается в том, что сервер аутентификации подписывает зашифрованный бюллетень `{таким образом не зная его содержимого}` набором ключей, выбор которых неоднозначен, так что по получившейся подписи невозможно установить какой именно набор ключей её подписал. Для корректности необходимо убедиться что наборы ключей сервера аутентификации, которыми он шифрует сообщения разных пользователей принадлежат одному семейству. Альтернативно в процессе слепого шифрования может подписываться не сам бюллетень, а косвенные данные или же использоваться публично известные хэш функции. [Так или иначе](https://ru.m.wikipedia.org/wiki/Протоколы_тайного_голосования#Протокол_Фудзиоки_—_Окамото_—_Оты) это возможно автоматизировать, так что разрешение этого пункта является делом техники.

## Улучшения

Потенциальным улучшением решающим часть проблем `{во всяком случае с мёртвыми душами}` может являться создание децентрализованной системы учёта голосов, основанной на принципе блокчейна. Однако данный подход внесёт уже свои особенности и проблемы.


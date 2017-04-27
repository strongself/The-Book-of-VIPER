# Дополнительные материалы Rambler&Co

#### Rambler.iOS V - V is for VIPER

В Rambler&Co периодически проводятся встречи iOS разработчиков. Одна из них была полностью посвящена VIPER - и стала основой для этой книги.

- Вступление ([Видео](http://www.youtube.com/watch?v=zjw6Md1mMjQ)) - [Егор Толстой](https://github.com/igrekde)
- VIPER a la Rambler ([Видео](http://www.youtube.com/watch?v=mEju4PyuCBM) | [Слайды](http://www.slideshare.net/Rambler-iOS/viper-a-la-rambler)) - [Сергей Крапивенский](https://github.com/serkrapiv)
- Кодогенерация и Генерамба ([Видео](http://www.youtube.com/watch?v=NXNiN9FaUnY) | [Слайды](http://www.slideshare.net/Rambler-iOS/viper-56423582)) - [Егор Толстой](https://github.com/igrekde)
- Переходы между модулями ([Видео](http://www.youtube.com/watch?v=XvAHqDvGqzE) | [Слайды](http://www.slideshare.net/Rambler-iOS/viper-56423732)) - [Вадим Смаль](https://github.com/CognitiveDisson)
- Сложные модули ([Видео](http://www.youtube.com/watch?v=4ZPQ_qotx4M) | [Слайды](http://www.slideshare.net/Rambler-iOS/viper-56423837)) - [Андрей Зарембо](https://github.com/AndreyZarembo)
- Разбиваем Massive View Controller ([Видео](http://www.youtube.com/watch?v=aVuIk6F2rFA) | [Слайды](http://www.slideshare.net/Rambler-iOS/massive-view-controller)) - [Александр Сычев](https://github.com/Brain89)
- Тестирование VIPER ([Видео](http://www.youtube.com/watch?v=1y2vxtD7b6g) | [Слайды](http://www.slideshare.net/Rambler-iOS/tdd-viper)) -[Станислав Цыганов](https://github.com/AlloyDev)
- VIPER и Swift ([Видео](http://www.youtube.com/watch?v=m4MYKzlqtH8) | [Слайды](http://www.slideshare.net/Rambler-iOS/viper-swift)) - [Валерий Попов](https://github.com/complexityclass)
- Секция вопросов и ответов ([Видео](http://www.youtube.com/watch?v=mFvAIcL4C_4)) - [Егор Толстой](https://github.com/igrekde), [Сергей Крапивенский](https://github.com/serkrapiv)

#### Rambler&IT

Теоретические материалы - это отлично, но одной из главных проблем, с которой мы столкнулись при знакомстве с VIPER, это отсутствие примеров его применения в приложениях сложнее обычного *Hello World*.

Мы постарались решить этот вопрос и выложили в Open Source приложение [Rambler&IT](https://github.com/rambler-digital-solutions/RamblerConferences). Его основные особенности:

- Разбито на три основных слоя: `Presentation`, `BusinessLogic`, `Core`.
- Слой `Presentation` целиком написан с использованием VIPER.
- Слой `BusinessLogic` написан с использованием Service Oriented Architecture.
- Слой `Core` написан с использованием концепции составных операций, вдохноновленной [сессией 226](https://developer.apple.com/videos/play/wwdc2015/226) WWDC 2015.
- Для Dependency Injection активно используется Typhoon.

Приложение получилось достаточно крупным и сложным и продолжает активно развиваться. На настоящий момент это наиболее полный открытый пример использования VIPER в боевом проекте.

#### Другие конференции

Помимо Rambler.iOS мы и на других конференциях рассказывали про использование VIPER.

- VIPER - или то, о чем все говорят, но никто не рассказывает ([Видео](https://www.youtube.com/watch?v=dGTdlNjh_5U) | [Слайды](https://speakerdeck.com/etolstoy/viper-ili-to-o-chiem-vsie-ghovoriat-no-nikto-nie-rasskazyvaiet)) - [Егор Толстой](https://github.com/igrekde)
- Чистая архитектура с VIPER ([Видео](https://www.youtube.com/watch?v=uS8zropcvdU) | [Слайды](http://www.slideshare.net/codefest/ss-60159923)) - [Сергей Крапивенский](https://github.com/serkrapiv)
- VIPER: наш взгляд на вопрос ([Видео](https://www.youtube.com/watch?v=WY63iqXXtG4) | [Слайды](http://www.slideshare.net/uamobile/viper-ua-mobile-2016)) - [Екатерина Коровкина]()

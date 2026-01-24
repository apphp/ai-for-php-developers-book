# Table of contents

## Введение

* [Титульный лист](README.md)
* [Оглавление](vvedenie/oglavlenie.md)
* [Вступление](vvedenie/vstuplenie.md)
* [Зачем PHP-разработчику AI](vvedenie/zachem-php-razrabotchiku-ai.md)
* [Экосистема ML в PHP](vvedenie/ekosistema-ml-v-php/README.md)
  * [Настройка среды для PHP](vvedenie/ekosistema-ml-v-php/nastroika-sredy-dlya-php/README.md)
    * [Установка напрямую](vvedenie/ekosistema-ml-v-php/nastroika-sredy-dlya-php/ustanovka-napryamuyu.md)
    * [Установка с Docker](vvedenie/ekosistema-ml-v-php/nastroika-sredy-dlya-php/ustanovka-s-docker.md)
    * [Примечания](vvedenie/ekosistema-ml-v-php/nastroika-sredy-dlya-php/primechaniya.md)
* [Как эта книга устроена](vvedenie/kak-eta-kniga-ustroena/README.md)
  * [Лицензия и авторские права](vvedenie/kak-eta-kniga-ustroena/licenziya-i-avtorskie-prava.md)
  * [Как контрибьютить](vvedenie/kak-eta-kniga-ustroena/kak-kontribyutit.md)
  * [История изменений книги](vvedenie/kak-eta-kniga-ustroena/istoriya-izmenenii-knigi.md)
* [Глоссарий](vvedenie/glossarii.md)
* [Что дальше](vvedenie/chto-dalshe.md)

## Часть I. Математический язык AI

* [Что такое модель в математическом смысле](chast-i.-matematicheskii-yazyk-ai/chto-takoe-model-v-matematicheskom-smysle.md)
* [Векторы, размерности и пространства признаков](chast-i.-matematicheskii-yazyk-ai/vektory-razmernosti-i-prostranstva-priznakov.md)
* [Расстояния и сходство](chast-i.-matematicheskii-yazyk-ai/rasstoyaniya-i-skhodstvo/README.md)
  * [Практические кейсы](chast-i.-matematicheskii-yazyk-ai/rasstoyaniya-i-skhodstvo/prakticheskie-keisy/README.md)
    * [Кейс 1: Сравнение объектов и пользователей](chast-i.-matematicheskii-yazyk-ai/rasstoyaniya-i-skhodstvo/prakticheskie-keisy/keis-1-sravnenie-obektov-i-polzovatelei.md)
    * [Кейс 2: Оценка релевантности объекта](chast-i.-matematicheskii-yazyk-ai/rasstoyaniya-i-skhodstvo/prakticheskie-keisy/keis-2-ocenka-relevantnosti-obekta.md)
    * [Кейс 3: Сравнение текстов](chast-i.-matematicheskii-yazyk-ai/rasstoyaniya-i-skhodstvo/prakticheskie-keisy/keis-3-sravnenie-tekstov.md)

## Часть II. Обучение как оптимизация

* [Ошибка, loss-функции и зачем они нужны](chast-ii.-obuchenie-kak-optimizaciya/oshibka-loss-funkcii-i-zachem-oni-nuzhny/README.md)
  * [Практические кейсы](chast-ii.-obuchenie-kak-optimizaciya/oshibka-loss-funkcii-i-zachem-oni-nuzhny/prakticheskie-keisy/README.md)
    * [Кейс 1: MSE и цена большого промаха](chast-ii.-obuchenie-kak-optimizaciya/oshibka-loss-funkcii-i-zachem-oni-nuzhny/prakticheskie-keisy/keis-1-mse-i-cena-bolshogo-promakha.md)
    * [Кейс 2. Выбор модели через loss-функцию](chast-ii.-obuchenie-kak-optimizaciya/oshibka-loss-funkcii-i-zachem-oni-nuzhny/prakticheskie-keisy/keis-2.-vybor-modeli-cherez-loss-funkciyu.md)
    * [Кейс 3. Log loss и уверенность классификатора](chast-ii.-obuchenie-kak-optimizaciya/oshibka-loss-funkcii-i-zachem-oni-nuzhny/prakticheskie-keisy/keis-3.-log-loss-i-uverennost-klassifikatora.md)
    * [Кейс 4. Одинаковая точность – разный log loss](chast-ii.-obuchenie-kak-optimizaciya/oshibka-loss-funkcii-i-zachem-oni-nuzhny/prakticheskie-keisy/keis-4.-odinakovaya-tochnost-raznyi-log-loss.md)
    * [Кейс 5. Обучение модели как минимизация ошибки](chast-ii.-obuchenie-kak-optimizaciya/oshibka-loss-funkcii-i-zachem-oni-nuzhny/prakticheskie-keisy/keis-5.-obuchenie-modeli-kak-minimizaciya-oshibki.md)
* [Линейная регрессия как базовая модель](chast-ii.-obuchenie-kak-optimizaciya/lineinaya-regressiya-kak-bazovaya-model/README.md)
  * [Практические кейсы](chast-ii.-obuchenie-kak-optimizaciya/lineinaya-regressiya-kak-bazovaya-model/prakticheskie-keisy/README.md)
    * [Кейс 1. Оценка стоимости квартиры по параметрам](chast-ii.-obuchenie-kak-optimizaciya/lineinaya-regressiya-kak-bazovaya-model/prakticheskie-keisy/keis-1.-ocenka-stoimosti-kvartiry-po-parametram.md)
    * [Кейс 2. Прогноз времени выполнения задачи разработчиком](chast-ii.-obuchenie-kak-optimizaciya/lineinaya-regressiya-kak-bazovaya-model/prakticheskie-keisy/keis-2.-prognoz-vremeni-vypolneniya-zadachi-razrabotchikom.md)
    * [Кейс 3. Прогноз потребления ресурсов сервера](chast-ii.-obuchenie-kak-optimizaciya/lineinaya-regressiya-kak-bazovaya-model/prakticheskie-keisy/keis-3.-prognoz-potrebleniya-resursov-servera.md)
    * [Кейс 4. Оценка вероятного чека клиента](chast-ii.-obuchenie-kak-optimizaciya/lineinaya-regressiya-kak-bazovaya-model/prakticheskie-keisy/keis-4.-ocenka-veroyatnogo-cheka-klienta.md)
    * [Кейс 5. Прогноз зарплаты по рынку](chast-ii.-obuchenie-kak-optimizaciya/lineinaya-regressiya-kak-bazovaya-model/prakticheskie-keisy/keis-5.-prognoz-zarplaty-po-rynku.md)
* [Градиентный спуск на пальцах](chast-ii.-obuchenie-kak-optimizaciya/gradientnyi-spusk-na-palcakh/README.md)
  * [Эксперименты с градиентным спуском](chast-ii.-obuchenie-kak-optimizaciya/gradientnyi-spusk-na-palcakh/eksperimenty-s-gradientnym-spuskom/README.md)
    * [Пример 1. Траектория параметра](chast-ii.-obuchenie-kak-optimizaciya/gradientnyi-spusk-na-palcakh/eksperimenty-s-gradientnym-spuskom/primer-1.-traektoriya-parametra.md)
    * [Пример 2. Влияние learning rate](chast-ii.-obuchenie-kak-optimizaciya/gradientnyi-spusk-na-palcakh/eksperimenty-s-gradientnym-spuskom/primer-2.-vliyanie-learning-rate.md)
    * [Пример 3. Плато и почти нулевой градиент](chast-ii.-obuchenie-kak-optimizaciya/gradientnyi-spusk-na-palcakh/eksperimenty-s-gradientnym-spuskom/primer-3.-plato-i-pochti-nulevoi-gradient.md)
    * [Пример 4. Batch и стохастический спуск](chast-ii.-obuchenie-kak-optimizaciya/gradientnyi-spusk-na-palcakh/eksperimenty-s-gradientnym-spuskom/primer-4.-batch-i-stokhasticheskii-spusk.md)

## Часть III. Классификация и вероятности

* [Вероятность как степень уверенности](chast-iii.-klassifikaciya-i-veroyatnosti/veroyatnost-kak-stepen-uverennosti/README.md)
  * [Практические кейсы](chast-iii.-klassifikaciya-i-veroyatnosti/veroyatnost-kak-stepen-uverennosti/prakticheskie-keisy/README.md)
    * [Кейс 1. Фильтр спама: вероятность ≠ решение](chast-iii.-klassifikaciya-i-veroyatnosti/veroyatnost-kak-stepen-uverennosti/prakticheskie-keisy/keis-1.-filtr-spama-veroyatnost-reshenie.md)
    * [Кейс 2. Медицинский тест: обновление уверенности (Байес)](chast-iii.-klassifikaciya-i-veroyatnosti/veroyatnost-kak-stepen-uverennosti/prakticheskie-keisy/keis-2.-medicinskii-test-obnovlenie-uverennosti-baies.md)
    * [Кейс 3. Многоклассовая классификация и softmax (RubixML)](chast-iii.-klassifikaciya-i-veroyatnosti/veroyatnost-kak-stepen-uverennosti/prakticheskie-keisy/keis-3.-mnogoklassovaya-klassifikaciya-i-softmax-rubixml.md)
    * [Кейс 4. Переуверенная модель как сигнал проблемы](chast-iii.-klassifikaciya-i-veroyatnosti/veroyatnost-kak-stepen-uverennosti/prakticheskie-keisy/keis-4.-pereuverennaya-model-kak-signal-problemy.md)
    * [Кейс 5. Обновление уверенности при новых данных](chast-iii.-klassifikaciya-i-veroyatnosti/veroyatnost-kak-stepen-uverennosti/prakticheskie-keisy/keis-5.-obnovlenie-uverennosti-pri-novykh-dannykh.md)
* [Логистическая регрессия](chast-iii.-klassifikaciya-i-veroyatnosti/logisticheskaya-regressiya/README.md)
  * [Практические кейсы](chast-iii.-klassifikaciya-i-veroyatnosti/logisticheskaya-regressiya/prakticheskie-keisy/README.md)
    * [Кейс 1. Логистическая регрессия для оттока клиентов](chast-iii.-klassifikaciya-i-veroyatnosti/logisticheskaya-regressiya/prakticheskie-keisy/keis-1.-logisticheskaya-regressiya-dlya-ottoka-klientov.md)
    * [Кейс 2. Подписка на рассылку](chast-iii.-klassifikaciya-i-veroyatnosti/logisticheskaya-regressiya/prakticheskie-keisy/keis-2.-podpiska-na-rassylku.md)
    * [Кейс 3. Спам или не спам](chast-iii.-klassifikaciya-i-veroyatnosti/logisticheskaya-regressiya/prakticheskie-keisy/keis-3.-spam-ili-ne-spam.md)
    * [Кейс 4. Клик по рекламе (CTR)](chast-iii.-klassifikaciya-i-veroyatnosti/logisticheskaya-regressiya/prakticheskie-keisy/keis-4.-klik-po-reklame-ctr.md)
    * [Кейс 5. Одобрение кредита](chast-iii.-klassifikaciya-i-veroyatnosti/logisticheskaya-regressiya/prakticheskie-keisy/keis-5.-odobrenie-kredita.md)
    * [Кейс 6. Фрод или нормальная транзакция](chast-iii.-klassifikaciya-i-veroyatnosti/logisticheskaya-regressiya/prakticheskie-keisy/keis-6.-frod-ili-normalnaya-tranzakciya.md)
    * [Кейс 7. Медицинский скрининг](chast-iii.-klassifikaciya-i-veroyatnosti/logisticheskaya-regressiya/prakticheskie-keisy/keis-7.-medicinskii-skrining.md)
    * [Кейс 8. Технический дефект оборудования](chast-iii.-klassifikaciya-i-veroyatnosti/logisticheskaya-regressiya/prakticheskie-keisy/keis-8.-tekhnicheskii-defekt-oborudovaniya.md)
* [Почему наивный Байес работает](chast-iii.-klassifikaciya-i-veroyatnosti/pochemu-naivnyi-baies-rabotaet/README.md)
  * [Практические кейсы](chast-iii.-klassifikaciya-i-veroyatnosti/pochemu-naivnyi-baies-rabotaet/prakticheskie-keisy/README.md)
    * [Кейс 1. Категориальные признаки и частоты](chast-iii.-klassifikaciya-i-veroyatnosti/pochemu-naivnyi-baies-rabotaet/prakticheskie-keisy/keis-1.-kategorialnye-priznaki-i-chastoty.md)
    * [Кейс 2. Спам-фильтр на словах (Bernoulli Naive Bayes)](chast-iii.-klassifikaciya-i-veroyatnosti/pochemu-naivnyi-baies-rabotaet/prakticheskie-keisy/keis-2.-spam-filtr-na-slovakh-bernoulli-naive-bayes.md)
    * [Кейс 3. Числовые признаки (Gaussian Naive Bayes)](chast-iii.-klassifikaciya-i-veroyatnosti/pochemu-naivnyi-baies-rabotaet/prakticheskie-keisy/keis-3.-chislovye-priznaki-gaussian-naive-bayes.md)

## Часть IV. Близость и структура данных

* [Алгоритм k-ближайших соседей и локальные решения](chast-iv.-blizost-i-struktura-dannykh/algoritm-k-blizhaishikh-sosedei-i-lokalnye-resheniya/README.md)
  * [Практические кейсы](chast-iv.-blizost-i-struktura-dannykh/algoritm-k-blizhaishikh-sosedei-i-lokalnye-resheniya/prakticheskie-keisy/README.md)
    * [Кейс 1. Классификация клиента по поведению](chast-iv.-blizost-i-struktura-dannykh/algoritm-k-blizhaishikh-sosedei-i-lokalnye-resheniya/prakticheskie-keisy/keis-1.-klassifikaciya-klienta-po-povedeniyu.md)
    * [Кейс 2. Регрессия: оценка цены квартиры](chast-iv.-blizost-i-struktura-dannykh/algoritm-k-blizhaishikh-sosedei-i-lokalnye-resheniya/prakticheskie-keisy/keis-2.-regressiya-ocenka-ceny-kvartiry.md)
    * [Кейс 3. Классификация с RubixML](chast-iv.-blizost-i-struktura-dannykh/algoritm-k-blizhaishikh-sosedei-i-lokalnye-resheniya/prakticheskie-keisy/keis-3.-klassifikaciya-s-rubixml.md)
* [Decision Trees и разбиение пространства](chast-iv.-blizost-i-struktura-dannykh/decision-trees-i-razbienie-prostranstva/README.md)
  * [Практические кейсы](chast-iv.-blizost-i-struktura-dannykh/decision-trees-i-razbienie-prostranstva/prakticheskie-keisy/README.md)
    * [Кейс 1. Учебное дерево решений на чистом PHP](chast-iv.-blizost-i-struktura-dannykh/decision-trees-i-razbienie-prostranstva/prakticheskie-keisy/keis-1.-uchebnoe-derevo-reshenii-na-chistom-php.md)
    * [Кейс 2. Классификация с помощью RubixML](chast-iv.-blizost-i-struktura-dannykh/decision-trees-i-razbienie-prostranstva/prakticheskie-keisy/keis-2.-klassifikaciya-s-pomoshyu-rubixml.md)
    * [Кейс 3. Когда дерево удобно в реальном продукте](chast-iv.-blizost-i-struktura-dannykh/decision-trees-i-razbienie-prostranstva/prakticheskie-keisy/keis-3.-kogda-derevo-udobno-v-realnom-produkte.md)

## ЧАСТЬ V. ТЕКСТ КАК МАТЕМАТИКА

* [Почему слова превращаются в числа](chast-v.-tekst-kak-matematika/pochemu-slova-prevrashayutsya-v-chisla.md)

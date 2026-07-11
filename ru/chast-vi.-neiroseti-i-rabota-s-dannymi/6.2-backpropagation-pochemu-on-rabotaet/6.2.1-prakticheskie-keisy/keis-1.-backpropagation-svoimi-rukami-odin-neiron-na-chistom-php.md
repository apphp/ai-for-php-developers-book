# Кейс 1. Backpropagation своими руками – один нейрон на чистом PHP

#### Цель кейса

В предыдущей главе мы реализовали перцептрон и увидели, как нейрон принимает решение:

```
входы → веса → сумма → активация → результат
```

Но перцептрон из предыдущего примера имел один существенный недостаток:

мы сами задавали веса.

Например:

```php
$w1 = 0.5;
$w2 = -0.3;
```

А настоящий машинный интеллект начинается с другого вопроса:

Как модель сама понимает, какие веса нужно изменить?

Ответ:

backpropagation.

В этом кейсе мы создадим минимальную нейросеть своими руками:

* без RubixML;
* без готовых алгоритмов;
* только PHP и математика.

Мы реализуем настоящий объект нейрона:

```
Neuron

- хранит веса
- принимает вход
- делает предсказание
- считает ошибку
- вычисляет градиент
- обновляет себя
```

***

## Сценарий: нейрон обнаружения фишинговых писем

Представим самый простой фильтр безопасности.

Есть один признак:

```
x = количество подозрительных факторов
```

Например:

| Письмо                 | x | Правильный ответ |
| ---------------------- | - | ---------------- |
| Обычное письмо         | 0 | 0                |
| Есть внешняя ссылка    | 1 | 1                |
| Ссылка + запрос пароля | 2 | 1                |

Наша задача:

научить нейрон понимать:

```
больше подозрительных факторов
            ↓
больше вероятность фишинга
```

***

## Модель одного нейрона

Наш нейрон:

```
              x

              |
              v

        +-----------+
        |  Neuron   |
        |           |
        | weight    |
        | bias      |
        +-----------+

              |
              v

          sigmoid

              |
              v

        probability
```

Математически:

\$$\
z = wx+b\
\$$

где:

* `x` — входной признак;
* `w` — вес;
* `b` — смещение;
* `z` — внутренний сигнал.

После этого применяется функция активации:

\$$\
a = sigmoid(z)\
\$$

На выходе получаем число:

```
0.0 ... 1.0
```

Например:

```
0.95 → почти точно фишинг

0.05 → обычное письмо
```

***

## Создаем класс Neuron

Начнем с структуры.

```php
class Neuron
{
    private float $weight;

    private float $bias;


    public function __construct()
    {
        $this->weight = mt_rand(-100, 100) / 100;

        $this->bias = 0;
    }
}
```

Теперь у нас есть объект, который представляет настоящий нейрон.

Он хранит свои параметры:

```
Neuron

weight
bias
```

Именно эти параметры будут изменяться во время обучения.

***

## Forward pass

Первый этап работы нейрона:

получить вход и сделать предсказание.

Добавим метод:

```php
public function forward(float $input): float
{
    $z = $this->weight * $input + $this->bias;

    return $this->sigmoid($z);
}
```

Теперь использование:

```php
$neuron = new Neuron();


$result = $neuron->forward(2);


echo $result;
```

Внутри происходит:

```
input = 2


2 * weight + bias


        ↓


      sigmoid


        ↓


prediction
```

***

## Функция активации

Добавляем sigmoid:

```php
private function sigmoid(float $x): float
{
    return 1 / (1 + exp(-$x));
}
```

Она переводит любое число в диапазон:

```
0 ... 1
```

Например:

```
z = 3

sigmoid(3)

= 0.95
```

***

## Нейрон должен помнить прошлый расчет

Для backpropagation нужно знать:

* какой был вход;
* какой был результат.

Добавляем свойства:

```php
private float $lastInput;

private float $lastOutput;
```

Меняем `forward()`:

```php
public function forward(float $input): float
{
    $this->lastInput = $input;


    $z = $this->weight * $input + $this->bias;


    $this->lastOutput =
        $this->sigmoid($z);


    return $this->lastOutput;
}
```

Теперь нейрон хранит историю:

```
forward()

input
 |
 |
 v

Neuron

 |
 |
 v

output


Neuron remembers:

lastInput
lastOutput
```

***

## Функция ошибки

Теперь нужно понять:

насколько нейрон ошибся.

Используем простую квадратичную ошибку:

\$$\
L=(a-y)^2\
\$$

где:

* `a` — предсказание нейрона;
* `y` — правильный ответ.

Например:

```
prediction:

0.6


correct:

1
```

Ошибка:

```
(0.6 - 1)^2

= 0.16
```

***

## Добавляем backward()

Теперь начинается самое важное.

Backpropagation отвечает на вопрос:

Насколько мой вес повлиял на ошибку?

Добавляем:

```php
private float $gradient;
```

Теперь метод:

```php
public function backward(float $target): void
{
    $dL_da =
        2 * ($this->lastOutput - $target);


    $da_dz =
        $this->sigmoidDerivative(
            $this->lastOutput
        );


    $dz_dw =
        $this->lastInput;


    $this->gradient =
        $dL_da *
        $da_dz *
        $dz_dw;
}
```

Разберем.

***

### Первая часть

```php
$dL_da
```

Отвечает:

```
насколько ошибка зависит
от результата нейрона
```

Формула:

\$$\
2(a-y)\
\$$

***

### Вторая часть

```php
$da_dz
```

Отвечает:

```
насколько изменение внутреннего сигнала
меняет выход
```

Для sigmoid:

\$$\
a(1-a)\
\$$

Код:

```php
private function sigmoidDerivative(float $a): float
{
    return $a * (1 - $a);
}
```

***

### Третья часть

```php
$dz_dw
```

Отвечает:

```
насколько вес влияет
на внутренний сигнал
```

Так как:

\$$\
z=wx+b\
\$$

получаем:

\$$\
\frac{dz}{dw}=x\
\$$

***

## Обновление веса

Теперь нейрон знает:

```
какая ошибка
и кто виноват
```

Осталось изменить вес.

Добавляем:

```php
public function update(float $learningRate): void
{
    $this->weight -=
        $learningRate * $this->gradient;
}
```

Это градиентный спуск:

\$$\
w=w-\alpha\frac{dL}{dw}\
\$$

***

## Полный класс Neuron

Теперь собираем всё:

```php
class Neuron
{
    private float $weight;

    private float $bias;


    private float $lastInput;

    private float $lastOutput;

    private float $gradient;


    public function __construct()
    {
        $this->weight =
            mt_rand(-100,100) / 100;

        $this->bias = 0;
    }


    public function forward(float $input): float
    {
        $this->lastInput = $input;


        $z =
            $this->weight * $input
            + $this->bias;


        $this->lastOutput =
            $this->sigmoid($z);


        return $this->lastOutput;
    }


    public function backward(float $target): void
    {
        $dL_da =
            2 * (
                $this->lastOutput
                - $target
            );


        $da_dz =
            $this->sigmoidDerivative(
                $this->lastOutput
            );


        $dz_dw =
            $this->lastInput;


        $this->gradient =
            $dL_da *
            $da_dz *
            $dz_dw;
    }


    public function update(float $lr): void
    {
        $this->weight -=
            $lr * $this->gradient;
    }


    private function sigmoid(float $x): float
    {
        return 1 / (1 + exp(-$x));
    }


    private function sigmoidDerivative(float $a): float
    {
        return $a * (1 - $a);
    }
}
```

***

## Обучение нейрона

Теперь у нас есть настоящая модель.

Создаем нейрон:

```php
$neuron = new Neuron();
```

Наш пример:

```
x = 2

target = 1
```

Цикл обучения:

```php
for ($epoch = 1; $epoch <= 100; $epoch++) {


    // Forward

    $prediction =
        $neuron->forward(2);



    // Backward

    $neuron->backward(1);



    // Update

    $neuron->update(0.1);



    echo
        "Epoch $epoch: "
        . $prediction
        . PHP_EOL;
}
```

***

## Что происходит на каждой эпохе

Каждый проход:

```
1. Вход

x = 2


↓

2. Нейрон делает прогноз


↓

3. Сравнение с правильным ответом


↓

4. Backpropagation считает ошибку


↓

5. Вес изменяется


↓

6. Следующий прогноз становится лучше
```

***

## Пример результата

После обучения:

```
Epoch 1:

prediction:
0.54


Epoch 20:

prediction:
0.81


Epoch 50:

prediction:
0.93


Epoch 100:

prediction:
0.97
```

Нейрон научился:

```
x = 2

↓

97% вероятность фишинга
```

***

## Что мы реализовали

В этом небольшом классе есть все основные элементы нейросети:

| Компонент           | Где реализован |
| ------------------- | -------------- |
| Вес                 | `$weight`      |
| Bias                | `$bias`        |
| Forward pass        | `forward()`    |
| Activation function | `sigmoid()`    |
| Loss gradient       | `backward()`   |
| Gradient descent    | `update()`     |

***

## Связь с настоящими ML-библиотеками

Наш класс:

```
Neuron

forward()

backward()

update()
```

В настоящих библиотеках:

```
Layer

forward()

backward()

Optimizer.step()
```

Разница только в масштабе.

Наш нейрон:

```
1 вход
1 вес
1 выход
```

Настоящая сеть:

```
миллионы входов
миллионы весов
миллионы градиентов
```

Но принцип тот же.

***

## Главное, что нужно запомнить

Backpropagation — это не отдельная магическая технология.

Это метод, который говорит нейрону:

“Посмотри на свою ошибку, вычисли свой вклад в нее и немного измени свои параметры”.

В следующем кейсе мы заменим один наш объект `Neuron` на настоящую многослойную сеть и посмотрим, как тот же механизм используется внутри RubixML.


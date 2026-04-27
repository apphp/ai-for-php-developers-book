# Кейс 4. MLP для нелинейного риска фишинга (RubixML)

Теперь усложним задачу.

Допустим:\
• сотрудники с очень низкой подготовкой – риск\
• сотрудники с очень высокой нагрузкой – тоже риск\
• средний уровень – безопасен

Это уже нелинейная зависимость.

⸻

Реализация на RubixML

```
use Rubix\ML\Datasets\Labeled;
use Rubix\ML\Classifiers\MLPClassifier;
use Rubix\ML\Transformers\ZScaleStandardizer;

$samples = [
    [1, 10],
    [5, 3],
    [10, 1],
    [3, 7],
];

$labels = ['risk', 'safe', 'risk', 'safe'];

$dataset = new Labeled($samples, $labels);
$dataset->apply(new ZScaleStandardizer());

$model = new MLPClassifier([8, 4]); // два скрытых слоя

$model->train($dataset);

$prediction = $model->predict([[4, 6]]);
```

### Что здесь важно

* каждый слой делает линейную комбинацию
* затем применяется нелинейность
* композиция создает сложную границу

<br>

Это уже реальная полносвязная сеть.








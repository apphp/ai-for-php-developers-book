# Кейс 4. Attention вместо среднего в RubixML

В Rubix ML можно написать кастомный Transformer.

Идея

Создать FeatureAggregatorTransformer, который:\
• принимает массив векторов\
• возвращает attention-взвешенный результат

Пример концепта:

```
use Rubix\ML\Transformers\Transformer;

class AttentionAggregator implements Transformer
{
    public function transform(array &$samples) : void
    {
        foreach ($samples as &$sample) {
            $sample = attention($sample, $sample, $sample)[0];
        }
    }
}
```

Теперь attention становится частью pipeline:

```
$pipeline = new Pipeline([
    new AttentionAggregator(),
    new Standardizer(),
], new LogisticRegression());
```

Это уже production-архитектура.


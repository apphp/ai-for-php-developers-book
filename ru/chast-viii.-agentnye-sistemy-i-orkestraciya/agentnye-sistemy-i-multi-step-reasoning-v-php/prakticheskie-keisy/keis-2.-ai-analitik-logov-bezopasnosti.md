# Кейс 2. AI-аналитик логов безопасности

Этот кейс особенно близок к задачам security awareness и phishing simulation.

#### Цель

Создать агента, который анализирует логи и ищет подозрительные события.

#### Сценарий

У нас есть лог:

```
login failed from 192.168.1.20
login failed from 192.168.1.20
login failed from 192.168.1.20
login success from 192.168.1.7
```

Пользователь спрашивает:

"Есть ли подозрительная активность?"

#### Multi-step reasoning

Агент выполняет шаги:

1 найти IP адреса\
2 посчитать количество попыток\
3 определить аномалии

#### Инструменты

```
extract_ips
count_frequency
detect_anomaly
```

#### Пример инструмента

```
class IpExtractor
{
    public function run(string $log): array
    {
        preg_match_all('/\d+\.\d+\.\d+\.\d+/', $log, $matches);

        return $matches[0];
    }
}
```

#### Подсчёт частоты

```
class FrequencyCounter
{
    public function run(array $ips): array
    {
        return array_count_values($ips);
    }
}
```

#### Пример reasoning

```
Step 1:
extract IPs

Observation:
192.168.1.20 repeated 3 times

Step 2:
check anomaly

Conclusion:
Possible brute force attack
```

Вывод

Такая архитектура позволяет создать AI-аналитика безопасности, который помогает SOC-командам быстро находить подозрительные паттерны.


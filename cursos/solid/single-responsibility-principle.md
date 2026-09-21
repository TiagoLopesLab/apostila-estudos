# Single Responsibility Principle

> [!NOTE] Referências
> Vídeo: https://www.youtube.com/watch?v=EWHTE1dQM4U
### Sobre

O _Single Responsibility Principle_, ou "Princípio da responsabilidade única" é o princípio do SOLID que diz que uma classe deve ter apenas um motivo para mudar. Ou seja, uma classe deve ter só uma responsabilidade e nada mais. Isso porque, quanto mais responsabilidades uma classe possui mais complexa e frequente é a sua manutenção.
### Problemática

Nó código abaixo, temos um exemplo de uma classe que realiza o processamento de um pedido. O método de processar o pedido realiza três funções: a checagem do inventário, o cálculo do valor total e o processamento do pagamento. Isso significa que, se um desses processos mudar, a classe terá que ser refatorada. Entre outras palavras, a classe `OrderProcessor` tem muitos motivos para mudar.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\Solid\SingleResponsibility;

class OrderProcessor
{
    public function processOrder(Order $order): void
    {
        $this->checkInventory($order);
        $this->calculateTotal($order);
        $this->processPayment($order);
    }

    private function checkInventory(Order $order): void
    {
        echo 'Checando o inventário' . PHP_EOL;
    }

    private function calculateTotal(Order $order): void
    {
        echo 'Calculando o total' . PHP_EOL;
    }

    private function processPayment(Order $order): void
    {
        echo 'Processando o pagamento' . PHP_EOL;
    }
}
```

Para complementar o exemplo, abaixo tem um exemplo de um script PHP que utiliza a classe de processamento do pedido. 
```php
<?php

declare(strict_types=1);

use Tiagolopes\Solid\SingleResponsibility\Order;
use Tiagolopes\Solid\SingleResponsibility\OrderProcessor;

require_once dirname(__DIR__) . '/vendor/autoload.php';

$order = new Order(
    uuid: '3424234234-sdasdas324234-32423dasfsd',
    amount: 150.99
);

$orderProcessor = new OrderProcessor();
$orderProcessor->processOrder($order);
```
### Solução

Uma das formas de resolver o problema respeitando o _Single Responsibility Principle_ é criar uma classe para cada uma dessas operações do processamento do pedido. O código abaixo é um exemplo disso, convertendo o método `checkInventory` da classe `OrderProcessor` para a classe `InventoryChecker`.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\Solid\SingleResponsibility;

class InventoryChecker
{
    public function check(Order $order): void
    {
        echo 'Checando o inventário' . PHP_EOL;
    }
}
```

Após converter todas as operações do processamento de um pedido em classes específicas, podemos alterar a classe base para que ela receba as outras classes (ou interfaces) no construtor e, na função `processOrder` apenas chamar o respectivo método de cada classe na ordem correta.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\Solid\SingleResponsibility;

readonly class OrderProcessor
{
    public function __construct(
        private InventoryChecker $inventoryChecker,
        private OrderCalculator $orderCalculator,
        private PaymentProcessor $paymentProcessor
    ) {
    }

    public function processOrder(Order $order): void
    {
        $this->inventoryChecker->check($order);
        $this->orderCalculator->calculate($order);
        $this->paymentProcessor->process($order);
    }
}
```

Para que o código cliente continue funcionando, é necessário passar as classes criadas na instanciação da classe `OrderProcessor`. Para não ter que fazer isso em cada local em que a classe é utilizada, uma possível solução é utilizar um contêiner de injeção de dependência. 
```php
<?php

declare(strict_types=1);

use Tiagolopes\Solid\SingleResponsibility\InventoryChecker;
use Tiagolopes\Solid\SingleResponsibility\Order;
use Tiagolopes\Solid\SingleResponsibility\OrderCalculator;
use Tiagolopes\Solid\SingleResponsibility\OrderProcessor;
use Tiagolopes\Solid\SingleResponsibility\PaymentProcessor;

require_once 'vendor/autoload.php';

$order = new Order(
    uuid: '3424234234-sdasdas324234-32423dasfsd',
    amount: 150.99
);

$orderProcessor = new OrderProcessor(
    inventoryChecker: new InventoryChecker(),
    orderCalculator: new OrderCalculator(),
    paymentProcessor: new PaymentProcessor()
);
$orderProcessor->processOrder($order);
```

Dessa forma, a classe `OrderProcessor` passa a ter somente um motivo para mudar: o processamento de pedido mudar, seja incluindo um novo processo, removendo um processo já existente ou alterando a ordem entre eles. Se algum processo específico mudar, basta alterar a classe específica desse processo.
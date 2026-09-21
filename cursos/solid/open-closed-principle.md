# Open Closed Principle

> [!NOTE] Referências
> Vídeo: https://www.youtube.com/watch?v=T6pF7BfAPIo

### Sobre

O _Open Closed Principle_, ou "Princípio aberto fechado" nos diz que uma classe deve estar aberta para extensão, mas fechada para modificação. Isso significa que podemos criar novos comportamentos sem precisar mexer na classe que já está funcionando.
### Problemática

No exemplo abaixo, temos a classe `Order`, responsável por representar um pedido. Ela possui dois métodos de aplicação de desconto: um para aplicar um desconto fixo e outra para aplicar um desconto percentual. Da forma que a classe está implementada, a cada novo desconto, será necessário incluir a lógica de desconto na classe, deixando ela cada vez maior e mais complexa.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\Solid\OpenClosed;

use DomainException;

class Order
{
    public function __construct(
        private(set) float $amount
    ) {
    }

    public function applyFixedDiscount(float $discount): void
    {
        if ($discount > $this->amount) {
            throw new DomainException(
	            'Desconto não pode ser maior que o valor do pedido'
			);
        }

        $this->amount -= $discount;
    }

    public function applyPercentageDiscount(float $percentage): void
    {
        if ($percentage > 100 || $percentage <= 0) {
            throw new DomainException(
	            'Informe um percentual maior que 0 até 100%'
			);
        }

        $this->amount -= ($this->amount * $percentage / 100);
    }
}
```

Para complementar o exemplo, o código abaixo possui um script PHP que utiliza a classe `Order`.
```php
<?php

declare(strict_types=1);

use Tiagolopes\Solid\OpenClosed\Order;

require_once dirname(__DIR__) . '/vendor/autoload.php';

$order = new Order(150);
$order->applyFixedDiscount(10);
echo $order->amount . PHP_EOL;

$order2 = new Order(593.80);
$order2->applyPercentageDiscount(25);
echo $order2->amount . PHP_EOL;
```
### Solução

Para resolver esse problema, podemos aplicar o princípio _Open Closed_ utilizando o padrão de projetos Strategy. Primeiro, criamos uma interface a qual todas as classes quem representam os descontos irão implementá-la.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\Solid\OpenClosed;

interface DiscountInterface
{
    public function apply(float $amount): float;
}
```

Ao invés de ter um método para cada desconto dentro de `Order`, podemos criar uma classe para cada desconto e implementar a interface criada. Assim todas as classes que representam um desconto terão um método em comum.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\Solid\OpenClosed;

use DomainException;

readonly class PercentageDiscount implements DiscountInterface
{
    private float $percentage;
    public function __construct(
        float $percentage
    ) {
        if ($percentage > 100 || $percentage <= 0) {
            throw new DomainException(
	            'Informe um percentual maior que 0 até 100%'
			);
        }

        $this->percentage = $percentage;
    }

    public function apply(float $amount): float
    {
        return $amount - ($amount * $this->percentage / 100);
    }
}
```

Por fim, a classe `Order` passa a receber uma classe que estende a interface `DiscountInterface` em seu construtor, sendo um parâmetro opcional já que, de acordo com a regra implementada, um pedido não precisa necessariamente ter um desconto.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\Solid\OpenClosed;

use DomainException;

class Order
{
    public function __construct(
        private float $amount,
        private ?DiscountInterface $discount = null
    ) {
    }

    public function setDiscount(DiscountInterface $discount): void
    {
        $this->discount = $discount;
    }

    public function getAmount(): float
    {
        if ($this->discount !== null) {
            $this->amount = $this->discount->apply($this->amount);
        }

        return $this->amount;
    }
}
```

O código do cliente fica da seguinte forma, com a classe referente ao desconto passada no construtor de `Order`.
```php
<?php

declare(strict_types=1);

use Tiagolopes\Solid\OpenClosed\FixedDiscount;
use Tiagolopes\Solid\OpenClosed\Order;
use Tiagolopes\Solid\OpenClosed\PercentageDiscount;

require_once dirname(__DIR__) . '/vendor/autoload.php';

$order = new Order(
    amount: 150,
    discount: new FixedDiscount(10)
);
echo $order->getAmount() . PHP_EOL;

$order2 = new Order(
    amount: 593.80,
    discount: new PercentageDiscount(25)
);
echo $order2->getAmount() . PHP_EOL;
```

Dessa forma, a cada novo desconto, é criada uma classe nova que estende a interface `DiscountInterface` e a classe `Order` não precisa ser modificada, tornando-a fechada para modificação.
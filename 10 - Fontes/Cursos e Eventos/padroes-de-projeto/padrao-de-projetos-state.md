# Padrão de projetos State

Vídeo: https://www.youtube.com/watch?v=OrCgWzpNszk

O **State** é um padrão de projeto comportamental que permite que um objeto altere seu comportamento quando seu estado interno muda. Parece como se o objeto mudasse de classe.

 ideia principal é que, em qualquer dado momento, há um número _finito_ de _estados_ que um programa possa estar. Dentro de qualquer estado único, o programa se comporta de forma diferente, e o programa pode ser trocado de um estado para outro instantaneamente. Contudo, dependendo do estado atual, o programa pode ou não trocar para outros estados. Essas regras de troca, chamadas _transições_.

### Problemática

No exemplo abaixo, a classe `Order` (que representa um pedido) possui um estado, que é definido como uma string e possui o valor inicial "placed". A classe possui métodos para alterar e recuperar o estado atual.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatternsPhp\Model;

class Order
{
    public function __construct(
        private string $state = 'placed'
    ) {
    }

    public function getState(): string
    {
        return $this->state;
    }

    public function setState(string $state): void
    {
        $this->state = $state;
    }
}
```

O principal problema disso é que, como os estados são representados por strings simples, é possível mudar para qualquer status, inclusive com nomes inexistentes, o que pode causar problemas na aplicação de regras de negócios para estados específicos. Além de ser difícil de definir quais são os status válidos.
```php
<?php

declare(strict_types=1);

use Tiagolopes\DesignPatternsPhp\Model\Order;

require_once dirname(__DIR__) . '/vendor/autoload.php';

$order = new Order();
$order->setState('pending');
$order->setState('deliveed');

if ($order->getState() === 'delivered') {
    echo 'Order delivered';
}
```

### Solução

O padrão **State** surge para solucionar esse problema. Primeiramente, vamos criar uma interface que define os métodos possíveis para transição de um estado para outro.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatternsPhp\Contracts;

use Tiagolopes\DesignPatternsPhp\Model\Order;

interface OrderStateInterface
{
    public function prepare(Order $order): void;

    public function deliver(Order $order): void;

    public function complete(Order $order): void;
}
```

Após isso, criamos classes que representam cada estado possível. Cada uma dessas classes irá estender a interface e implementar apenas os métodos que correspondam a uma transição de estado permitida de acordo com as regras de negócio.

No exemplo abaixo, quando o pedido (`Order`) está com o estado "recebido" (`OrderPlaced`) ele pode apenas transitar para o estado "em progresso" (`OrderInProgress`). Logo, apenas o método `prepare`, que corresponde a essa transição, é implementado. Os outros métodos apenas lançam uma exceção, isso porque não é permitido transitar para os outros estados a partir deste, segundo as regras de negócio da aplicação. 
```php
<?php

namespace Tiagolopes\DesignPatternsPhp\Model\OrderStates;

use DomainException;
use Tiagolopes\DesignPatternsPhp\Contracts\OrderStateInterface;
use Tiagolopes\DesignPatternsPhp\Model\Order;

class OrderPlaced implements OrderStateInterface
{
    public function prepare(Order $order): void
    {
        $order->setState(new OrderInProgress());
    }

    public function deliver(Order $order): void
    {
        throw new DomainException('Order has not been prepared.');
    }

    public function complete(Order $order): void
    {
        throw new DomainException('Order has not been prepared.');
    }
}
```

Nesse outro exemplo, quando um produto foi enviado, ele só pode assumir o estado "concluído" (`OrderCompleted`), logo os demais métodos apenas lançam uma `DomainException`.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatternsPhp\Model\OrderStates;

use DomainException;
use Tiagolopes\DesignPatternsPhp\Contracts\OrderStateInterface;
use Tiagolopes\DesignPatternsPhp\Model\Order;

class OrderShipped implements OrderStateInterface
{
    public function prepare(Order $order): void
    {
        throw new DomainException('Order has already been shipped.');
    }

    public function deliver(Order $order): void
    {
        throw new DomainException('Order has already been shipped.');
    }

    public function complete(Order $order): void
    {
        $order->setState(new OrderCompleted());
    }
}
```

Por fim, a classe de pedido deve receber o estado inicial no construtor e ter um método para cada transição de estado. Cada método chama a respectiva função do estado atual, transitando estre estados de uma forma controlada.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatternsPhp\Model;

use Tiagolopes\DesignPatternsPhp\Contracts\OrderStateInterface;
use Tiagolopes\DesignPatternsPhp\Model\OrderStates\OrderPlaced;

class Order
{
    public function __construct(
        private OrderStateInterface $state = new OrderPlaced()
    ) {
    }

    public function getState(): OrderStateInterface
    {
        return $this->state;
    }

    public function setState(OrderStateInterface $state): void
    {
        $this->state = $state;
    }

    public function prepare(): void
    {
        $this->state->prepare($this);
    }

    public function deliver(): void
    {
        $this->state->deliver($this);
    }

    public function complete(): void
    {
        $this->state->complete($this);
    }
}
```

Dessa forma, ao instanciarmos um pedido (`Order`), temos acesso a todos os métodos de transição de estado e, se tentarmos fazer uma transição indevida, será retornada uma exceção.
```php
<?php

declare(strict_types=1);

use Tiagolopes\DesignPatternsPhp\Model\Order;

require_once dirname(__DIR__) . '/vendor/autoload.php';

$order = new Order();
$order->prepare();
$order->deliver();
$order->complete();
```

Com isso, além de termos todos os estados possíveis mapeados, também temos o controle das transições possíveis de cada estado.

---
### Contéudos semelhantes

- Simple Factory: [[padrao-de-projetos-simple-factory]]
- Adapter: [[padrao-de-projetos-adapter]]
- Strategy: [[padrao-de-projetos-strategy]]
- Facade: [[padrao-de-projetos-facade]]
- Observer: [[padrao-de-projetos-observer]]
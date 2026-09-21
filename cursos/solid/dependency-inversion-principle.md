# Dependency Inversion Principle

> [!NOTE] Referências
> Vídeo: https://www.youtube.com/watch?v=s8g32ePcPps
### Sobre

O *Dependency Inversion Principle*, ou "Princípio de inversão de dependência" nos diz que uma classe não deve depender de uma implementação concreta, mas sim de uma interface.
### Problemática

Considere o exemplo abaixo. Nele, temos uma classe que realiza o processamento de um pedido utilizando o Stripe como gateway de pagamentos.
```php
<?php

namespace Tiagolopes\Solid\DependencyInversion;

use Stripe\StripeClient;

readonly class OrderProcessorService
{
    public function __construct(
        private StripeClient $client
    ) {
    }

    public function process(Order $order): void
    {
        $this->client->charges
            ->create(['amount' => $order->amount]);
    }
}
```

A classe `OrderProcessorService` é dependente da classe `StripeClient`, que provém de uma biblioteca externa a qual não temos nenhum controle. Isso fere o princípio de Inversão de Dependência visto que, se o pacote do Stripe receber atualizações, pode acabar afetando essa classe de serviço e qualquer outra classe que dependa do Stripe.
### Solução

Podemos resolver isso aplicando o princípio de Inversão de Dependência juntamente com o Padrão de projetos Adapter. Para isso, podemos criar uma interface que represente um Gateway de pagamentos.
```php
<?php

namespace Tiagolopes\Solid\DependencyInversion\Contracts;

interface PaymentGatewayInterface
{
    public function pay(int $amount): void;
}
```

Não podemos fazer com que a classe `StripeClient` estenda a interface criada visto que é uma dependência externa, mas podemos criar uma classe adaptadora que estende essa interface e adapta o método implementado para a biblioteca externa do Stripe.
```php
<?php

namespace Tiagolopes\Solid\DependencyInversion;

use Stripe\StripeClient;
use Tiagolopes\Solid\DependencyInversion\Contracts\PaymentGatewayInterface;

readonly class StripePaymentGateway implements PaymentGatewayInterface
{
    private StripeClient $client;
    public function __construct(?StripeClient $client)
    {
        $this->client = $client ?? new StripeClient();
    }

    public function pay(int $amount): void
    {
        $this->client->charges
            ->create(['amount' => $amount]);
    }
}
```

Dessa forma, podemos atualizar o service para que ele dependa da interface e não da classe concreta.
```php
<?php

namespace Tiagolopes\Solid\DependencyInversion;

use Tiagolopes\Solid\DependencyInversion\Contracts\PaymentGatewayInterface;

readonly class OrderProcessorService
{
    public function __construct(
        private PaymentGatewayInterface $gateway
    ) {
    }

    public function process(Order $order): void
    {
        $this->gateway->pay($order->amount);
    }
}
```

Assim, se quisermos implementar uma nova forma de pagamento, com Mercado Pago, por exemplo, basta criarmos uma nova classe adaptadora que estenda da mesma interface.
```php
<?php

namespace Tiagolopes\Solid\DependencyInversion;

use Tiagolopes\Solid\DependencyInversion\Contracts\PaymentGatewayInterface;

class MercadoPagoPaymentGateway implements PaymentGatewayInterface
{
    public function pay(int $amount): void
    {
        // Implementação via mercado pago utilizando uma lib externa
    }
}
```
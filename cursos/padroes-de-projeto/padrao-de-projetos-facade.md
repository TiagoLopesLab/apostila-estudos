
# Padrão de Projetos Facade

> [!NOTE] Link do vídeo
> https://www.youtube.com/watch?v=4Aq9UHQ5f5Y

### Sobre

O padrão _Facade_ consiste em abstrair uma lógica complexa em uma classe de "faixada", fazendo com que o código do "cliente" chame somente a classe de faixada, deixando o código mais simples e organizado.

### Problemática

O código abaixo possui um script em PHP que realiza o processamento de um pedido e, para isso, chama quatro services diferentes. Sempre que o processamento de pagamento mudar, será necessário alterar esse script. Isso não é necessariamente um problema, mas imagine que esse arquivo na verdade é um Controller de uma rota que realiza a compra de um produto em um marketplace. O Controller não é responsável pelas regras de negócio, mas terá que ser modificado sempre que a lógica de processamento de compra sofrer alterações.
```php
<?php

require_once dirname(__DIR__) . '/vendor/autoload.php';

use Tiagolopes\DesignPatternsPhp\Service\InventoryManager;
use Tiagolopes\DesignPatternsPhp\Service\Notifier;
use Tiagolopes\DesignPatternsPhp\Service\PaymentProcessor;
use Tiagolopes\DesignPatternsPhp\Service\ShippingService;

if (count($argv) < 5) {
    echo 'Required arguments missing' . PHP_EOL;
    exit(1);
}

$productId = (int) $argv[1];
$quantity = (int) $argv[2];
$amount = (float) $argv[3];
$userEmail = $argv[4];

$paymentProcessor = new PaymentProcessor();
$paymentProcessor->processPayment($amount);

$notifier = new Notifier();
$notifier->sendConfirmation($userEmail);

$inventoryManager = new InventoryManager();
$inventoryManager->updateStock($productId, $quantity);

$shippingService = new ShippingService();
$shippingService->initiateShipping($productId, $quantity);
```
### Solução

Para resolvermos isso utilizando o padrão _Facade_, basta criarmos uma classe de "fachada" que realiza todas as etapas do processo de compra, recebendo todos os _services_ necessários em seu construtor.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatternsPhp\Service\Facade;

use Tiagolopes\DesignPatternsPhp\OrderDto;
use Tiagolopes\DesignPatternsPhp\Service\InventoryManager;
use Tiagolopes\DesignPatternsPhp\Service\Notifier;
use Tiagolopes\DesignPatternsPhp\Service\PaymentProcessor;
use Tiagolopes\DesignPatternsPhp\Service\ShippingService;

readonly class OrderFacade
{
    public function __construct(
        private PaymentProcessor $paymentProcessor,
        private Notifier $notifier,
        private InventoryManager $inventoryManager,
        private ShippingService $shippingService
    ) {
    }

    public function processOrder(OrderDto $order): void
    {
        $this->paymentProcessor->processPayment($order->amount);
        $this->notifier->sendConfirmation($order->userEmail);
        $this->inventoryManager->updateStock($order->productId, $order->quantity);
        $this->shippingService->initiateShipping(
	        $order->productId,
	        $order->quantity
		);
    }
}
```

Essa etapa não é essencial, mas para mapearmos todos os parâmetros recebidos pelo Controller, podemos criar um DTO (_Data Transfer Object_). Dessa forma, a classe de "fachada" sabe de todos os parâmetros que ela pode receber.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatternsPhp;

readonly class OrderDto
{
    private function __construct(
        public int $productId,
        public int $quantity,
        public float $amount,
        public string $userEmail
    ) {
    }

    public static function fromArray(array $data): self
    {
        return new self(
            productId: $data['product_id'],
            quantity: $data['quantity'],
            amount: $data['amount'],
            userEmail: $data['user_email']
        );
    }
}
```

Por fim, alteramos o cliente (o Controller, ou no nosso caso, o script PHP) para que ele utilize somente a classe de "fachada". Logo, se for necessário alterar o processo de compra, será alterado na classe `OrderFacade` ou no service referente ao processo que foi alterado.
```php
<?php

require_once dirname(__DIR__) . '/vendor/autoload.php';

use Tiagolopes\DesignPatternsPhp\OrderDto;
use Tiagolopes\DesignPatternsPhp\Service\Facade\OrderFacade;
use Tiagolopes\DesignPatternsPhp\Service\InventoryManager;
use Tiagolopes\DesignPatternsPhp\Service\Notifier;
use Tiagolopes\DesignPatternsPhp\Service\PaymentProcessor;
use Tiagolopes\DesignPatternsPhp\Service\ShippingService;

if (count($argv) < 5) {
    echo 'Required arguments missing' . PHP_EOL;
    exit(1);
}

$order = [
    'product_id' => (int) $argv[1],
    'quantity' => (int) $argv[2],
    'amount' => (float) $argv[3],
    'user_email' => $argv[4],
];

$orderFacade = new OrderFacade(
    paymentProcessor: new PaymentProcessor(),
    notifier: new Notifier(),
    inventoryManager: new InventoryManager(),
    shippingService: new ShippingService()
);
$orderFacade->processOrder(OrderDto::fromArray($order));
```

---
### Conteúdos semelhantes

- Simple Factory: [[padrao-de-projetos-simple-factory]]
- Strategy: [[padrao-de-projetos-strategy]]
- State: [[padrao-de-projetos-state]]
- Adapter: [[padrao-de-projetos-adapter]]
- Observer: [[padrao-de-projetos-observer]]
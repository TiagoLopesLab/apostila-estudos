# Padrão de Projetos Observer

> [!NOTE] Vídeo
> https://www.youtube.com/watch?v=mv9JxI85Ac8
### Sobre

O Padrão de Projetos _Observer_ consiste em classes chamadas de "observadoras" e uma classe que está sendo observada. Quando um atributo dessa classe muda (e ele "interessa" para as classes observadoras), a própria notifica as classes observadoras de que aquele valor mudou.
### Problemática

No código abaixo, há uma classe chamada "Bitcoin" que possui apenas o atributo _price_ e métodos para atualizar e devolver esse valor.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatternsPhp\Model;

class Bitcoin
{
    private float $price;

    public function __construct()
    {
        $this->price = 0;
    }

    public function setPrice(float $price): void
    {
        if ($this->price !== $price) {
            $this->price = $price;
        }
    }

    public function getPrice(): float
    {
        return $this->price;
    }
}
```

No script PHP abaixo, a classe `Bitcoin` é instanciada e o preço é atualizado com base no retorno de uma API externa. A alteração do preço do Bitcoin "interessa" três outras classes, que precisam realizar ações quando esse evento ocorre. Por conta disso, é necessário fazer um `if` para verificar se o preço mudou e chamar todas as classes interessadas nesse evento.
```php
<?php

declare(strict_types=1);

require_once dirname(__DIR__) . '/vendor/autoload.php';

use Tiagolopes\DesignPatternsPhp\API\BinanceAPI;
use Tiagolopes\DesignPatternsPhp\Model\Bitcoin;
use Tiagolopes\DesignPatternsPhp\Service\BitcoinPriceLogger;
use Tiagolopes\DesignPatternsPhp\Service\InvestorNotifier;
use Tiagolopes\DesignPatternsPhp\Service\NewsPlatform;

$bitcoin = new Bitcoin();
$binanceApi = new BinanceAPI();

$lastPrice = $bitcoin->getPrice();
$newPrice = $binanceApi->getBitcoinLastPrice();
$bitcoin->setPrice($newPrice);

if ($lastPrice !== $newPrice) {
    $bitcoinPriceLogger = new BitcoinPriceLogger();
    $bitcoinPriceLogger->log($newPrice);

    $investorNotifier = new InvestorNotifier();
    $investorNotifier->notifier($newPrice);

    $newsPlatform = new NewsPlatform();
    $newsPlatform->sendNews($newPrice);
}
```
### Solução

Para deixar o código mais organizado, podemos aplicar o Padrão de Projetos _Observer_ e, para isso, primeiro precisamos que as classes "observadoras" tenham um método em comum que será chamado no momento da notificação pela classe observada. Portanto, podemos criar uma interface que será implementada por cada classe observadora.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatternsPhp\Contracts;

interface BitcoinPriceObserverInterface
{
    public function update(float $price): void;
}
```

Assim, todas as classes que dependem daquela informação passam a ter um método em comum que utiliza essa informação vinda da notificação para o seu fim específico.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatternsPhp\Service;

use Tiagolopes\DesignPatternsPhp\Contracts\BitcoinPriceObserverInterface;

class BitcoinPriceLogger implements BitcoinPriceObserverInterface
{
    public function update(float $price): void
    {
        echo "Logging new Bitcoin Price: $price" . PHP_EOL;
    }
}
```

Para que a classe observada possa enviar uma notificação para as classes observadoras, podemos criar um array que armazena a lista de observadores, juntamente com um método para adicionar itens na lista e notificar as classes da lista, percorrendo todas elas e chamando o método que elas possuem em comum. Dessa forma, basta incluir o método no momento exato em que a notificação deve ser disparada. 
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatternsPhp\Model;

use InvalidArgumentException;
use Tiagolopes\DesignPatternsPhp\Contracts\BitcoinPriceObserverInterface;

class Bitcoin
{
    private float $price;
    /** @var BitcoinPriceObserverInterface[] $observers */
    private array $observers;

    public function __construct()
    {
        $this->price = 0.0;
        $this->observers = [];
    }

    public function setPrice(float $price): void
    {
        if ($this->price !== $price) {
            $this->price = $price;

            $this->notifyObservers();
        }
    }

    public function getPrice(): float
    {
        return $this->price;
    }

    /** @param BitcoinPriceObserverInterface[] $observers */
    public function addObservers(array $observers): void
    {
        foreach ($observers as $observer) {
            if (!$observer instanceof BitcoinPriceObserverInterface) {
                throw new InvalidArgumentException('Invalid observer!');
            }

            $this->observers[] = $observer;
        }
    }

    private function notifyObservers(): void
    {
        foreach ($this->observers as $observer) {
            $observer->update($this->price);
        }
    }
}
```

Por fim, alteramos no código do cliente e apenas incluímos as classes observadoras na lista para que elas sejam notificadas no momento certo.
```php
<?php

declare(strict_types=1);

require_once dirname(__DIR__) . '/vendor/autoload.php';

use Tiagolopes\DesignPatternsPhp\API\BinanceAPI;
use Tiagolopes\DesignPatternsPhp\Model\Bitcoin;
use Tiagolopes\DesignPatternsPhp\Service\BitcoinPriceLogger;
use Tiagolopes\DesignPatternsPhp\Service\InvestorNotifier;
use Tiagolopes\DesignPatternsPhp\Service\NewsPlatform;

$bitcoin = new Bitcoin();
$binanceApi = new BinanceAPI();

$bitcoin->addObservers([
    new BitcoinPriceLogger(),
    new InvestorNotifier(),
    new NewsPlatform()
]);

$newPrice = $binanceApi->getBitcoinLastPrice();
$bitcoin->setPrice($newPrice);
```

---
### Conteúdos semelhantes

- Simple Factory: [[padrao-de-projetos-simple-factory]]
- Strategy: [[padrao-de-projetos-strategy]]
- State: [[padrao-de-projetos-state]]
- Adapter: [[padrao-de-projetos-adapter]]
- Facade: [[padrao-de-projetos-facade]]
- Template Method: [[padrao-de-projetos-template-method]]
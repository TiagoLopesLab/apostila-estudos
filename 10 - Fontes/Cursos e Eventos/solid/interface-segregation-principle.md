# Interface Segregation Principle

> [!NOTE] Referências
> Vídeo: https://www.youtube.com/watch?v=jHbI9ej5O1Y
### Sobre

O *Interface Segregation Principle*, ou "Princípio da segregação de interface" nos diz que Nenhuma classe deve ser forçada a depender de métodos que não utiliza.
### Problemática

Considere que, em um projeto fictício é necessário implementar o processamento de diversas formas de pagamentos diferentes, como: Cartão de Crédito, Pix e PayPal. Visto que todas essas entidades teriam uma função em comum (processar pagamento) foi criada uma interface chamada `PaymentMethodInterface` a qual todas as classes que representam uma forma de pagamento implementariam.
```php
<?php

namespace Tiagolopes\Solid\InterfaceSegregation\Contracts;

interface PaymentMethodInterface
{
    public function pay(): void;
}
```

Dessa forma cada uma das classes de pagamento vão implementar essa interface, como a classe `Pix` no exemplo abaixo.
```php
<?php

namespace Tiagolopes\Solid\InterfaceSegregation;

use Tiagolopes\Solid\InterfaceSegregation\Contracts\PaymentMethodInterface;

class Pix implements PaymentMethodInterface
{
    public function pay(): void
    {
        echo 'Pagamento via Pix' . PHP_EOL;
    }
}
```

E assim foi feito para as outras classes. Até o momento, não há nada de errado com a implementação. 

Até que surgiu uma demanda nova em que o sistema deve possibilitar gerar um QR Code para pagamentos via Pix. Portanto, para o método não ficar "solto" na classe `Pix`, ele foi incluído na interface `PaymentMethodInterface` e foi implementado dentro de `Pix`.
```php
<?php

namespace Tiagolopes\Solid\InterfaceSegregation\Contracts;

interface PaymentMethodInterface
{
    public function pay(): void;
    public function generateQrCode(): void;
}
```

O problema disso, é que todas as outras classes que implementam essa interface vão precisar implementar o novo método, mesmo que ela não irá fazer uso dele, já que essa funcionalidade seria exclusivamente para o método de pagamento Pix. Uma abordagem muito comum seria implementar o método contendo apenas o lançamento de uma exceção nos casos em que a classe não iria utilizar o novo método. Dessa forma, o contrato não era violado.
```php
<?php

namespace Tiagolopes\Solid\InterfaceSegregation;

use Tiagolopes\Solid\InterfaceSegregation\Contracts\PaymentMethodInterface;

class CreditCard implements PaymentMethodInterface
{
    public function pay(): void
    {
        echo 'Pagamento via cartão de crédito' . PHP_EOL;
    }

    public function generateQrCode(): void
    {
        throw new \DomainException('Method not allowed');
    }
}
```

Porém, conforme a interface vai tendo novos métodos fica inviável implementar em todas as classes incluindo as que não irão utilizá-lo.
### Solução

Para resolver esse problema, basta "quebrarmos" essa interface em duas interfaces para garantir que uma classe só irá estender as interfaces as quais possuem os métodos que ela irá utilizar, de fato.

No exemplo acima, o método que "estava sobrando" era o `generateQrCode`, que era usado unicamente pela classe `Pix`. Nesse caso, podemos criar uma nova interface chamada `QrCodeGenerableInterface` com apenas esse método.
```php
<?php

namespace Tiagolopes\Solid\InterfaceSegregation\Contracts;

interface QrCodeGenerableInterface
{
    public function generateQrCode(): void;
}
```

Então, podemos remover o método da interface `PaymentMethodInterface` bem como das classes que não utilizam esse método. E para a classe `Pix`, basta implementarmos uma segunda interface, que é essa `QrCodeGenerableInterface`, a qual possui o método que a classe `Pix` precisa.
```php
<?php

namespace Tiagolopes\Solid\InterfaceSegregation;

use Tiagolopes\Solid\InterfaceSegregation\Contracts\PaymentMethodInterface;
use Tiagolopes\Solid\InterfaceSegregation\Contracts\QrCodeGenerableInterface;

class Pix implements PaymentMethodInterface, QrCodeGenerableInterface
{
    public function pay(): void
    {
        echo 'Pagamento via Pix' . PHP_EOL;
    }

    public function generateQrCode(): void
    {
        echo 'Gerando o QR Code' . PHP_EOL;
    }
}
```

Assim, todas as classes estão implementando apenas os métodos que elas necessitam e o princípio está sendo respeitado.
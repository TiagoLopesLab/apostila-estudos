# Evite Getters e Setters

> [!NOTE] Referências
> Vídeo: https://www.youtube.com/watch?v=PXHqooNlkVM
## Introdução

Essa regra do *Object Calisthenics* nos diz que devemos evitar ao máximo o uso de métodos *getters* e *setters*. Um método *getter* retorna o valor atual de uma propriedade da classe enquanto o *setter* atualiza esse valor de forma deliberada. O documento original cita os principais motivos pelo qual deve se evitar o uso desses métodos. Alguns deles são:
- Fere o encapsulamento;
- Separa o comportamento da entidade;
- Fere o princípio *Tell, Don't ask* ("Mande, não pergunte").
## Problemática

O código abaixo exemplifica o uso de *getters* e *setters*. A classe `Product` possui o atributo `$price` e foram criados os métodos `setPrice` e `getPrice` para atualizar e recuperar o valor respectivamente.
```php
<?php

namespace Tiagolopes\ObjectCalisthenics\GettersAndSetters;

class Product
{
    public function __construct(
        private float $price
    ) {
    }

    public function setPrice(float $price): void
    {
        $this->price = $price;
    }

    public function getPrice(): float
    {
        return $this->price;
    }
}
```

No exemplo abaixo, a classe é utilizada pelo código cliente e ele estabelece uma condição para aplicar um determinado desconto para o produto. O problema é que essa lógica de desconto certamente é uma regra de negócio dessa aplicação fictícia e pertence ao produto, mas está fora da classe `Product`. Se essa regra mudar, o desenvolvedor terá que lembrar de alterá-la no arquivo cliente e onde mais essa classe for utilizada. Além disso, o método `setPrice` permite que o preço seja alterado livremente por qualquer arquivo que utilize essa classe, o que oferece um grande risco de uso indevido.
```php
<?php

require_once dirname(__DIR__) . '/vendor/autoload.php';

use Tiagolopes\ObjectCalisthenics\GettersAndSetters\Product;

$product = new Product(49.9);

$discount = 10;
$currentPrice = $product->getPrice();

// Regras de negócio no código cliente
if ($currentPrice > 50) {
    $product->setPrice($currentPrice - $discount);
}
```
## Solução

Para solucionarmos esse problema, podemos implementar a regra de desconto dentro da própria classe `Product`. Para isso, foi criada o método `applyDiscount` que aplica a regra que era feita no código cliente. Dessa forma, podemos remover o método `setPrice` já que a única mudança no preço até então era proveniente do desconto. Se tiver alguma outra lógica para a alteração do preço, ela também deveria ser incluída na própria classe. Além disso, o PHP moderno permite deixar a propriedade privada somente para alteração, dessa forma é possível consultar diretamente a variável e o método `getPrice` também não se faz mais necessário.
```php
<?php

namespace Tiagolopes\ObjectCalisthenics\GettersAndSetters;

class Product
{
    public function __construct(
        private(set) float $price
    ) {
    }

    public function applyDiscount(float $discount): float
    {
        if ($this->price > 50) {
            $this->price -= $discount;
        }

        return $this->price;
    }
}
```

O arquivo cliente agora chama o método de aplicar o desconto e não atualiza diretamente o preço do produto, respeitando o encapsulamento.
```php
<?php

require_once dirname(__DIR__) . '/vendor/autoload.php';

use Tiagolopes\ObjectCalisthenics\GettersAndSetters\Product;

$product = new Product(49.9);

$discount = 10;
echo $product->applyDiscount($discount);
```

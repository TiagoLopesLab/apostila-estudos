
# Padrão de projetos Strategy

Vídeo: https://youtu.be/DzlXwgsc_AU?si=Rl4hA6GZ84PgCBNH

O padrão Strategy sugere que o usuário pegue uma classe que faz algo específico em diversas maneiras diferentes e extraia todos esses algoritmos para classes separadas chamadas estratégias. A classe original, chamada contexto, deve ter um campo para armazenar uma referência para um dessas estratégias. O contexto delega o trabalho para um objeto estratégia ao invés de executá-lo por conta própria.

Desta forma o contexto se torna independente das estratégias concretas, então você pode adicionar novos algoritmos ou modificar os existentes sem modificar o código do contexto ou outras estratégias.

### Problemática

O exemplo abaixo contém a implementação de uma classe que calcula impostos com base no tipo de imposto e valor. É feito um `if` para cada tipo de imposto para aplicar o calculo específico desse imposto. Isso fere o princípio _Open-closed_ do _SOLID_ visto que, para cada nova taxa, será adicionado um novo `if` nesse método.
```php
<?php  
  
namespace Tiagolopes\DesignPatternsPhp\Service;  
  
use InvalidArgumentException;  
  
class TaxCalculator  
{  
    /**  
     * @throws InvalidArgumentException  
     */  
    public function calculate(string $taxType, float $amount): float  
    {  
        if ($taxType == 'ICMS') {  
            return ($amount * 4) / 100;
        }  
        if ($taxType == 'ISS') {  
            return ($amount * 11) / 100;
        }  
        if ($taxType == 'IPI') {  
            return ($amount * 15) / 100; 
        }  
        throw new InvalidArgumentException("Tax Type not supported");
    }
}
```

```php
// Arquivo que utiliza a classe TaxCalculator
<?php  
  
declare(strict_types=1);  
  
require_once __DIR__ . '/../vendor/autoload.php';  
  
use Tiagolopes\DesignPatternsPhp\Service\TaxCalculator;  
  
$taxCalculator = new TaxCalculator();  
var_dump($taxCalculator->calculate('ICMS', 1000));
```

### Solução

Para resolver esse problema, podemos aplicar o padrão Strategy. Primeiro, podemos criar uma interface que diz que cada imposto deverá implementar o método `calculate` de acordo com o cálculo específico dele.
```php
<?php  
  
namespace Tiagolopes\DesignPatternsPhp\Contracts;  
  
interface TaxTypeInterface  
{  
    public function calculate(float $amount): float;  
}
```

Para cada tipo de imposto, devemos implementar uma classe que implementa o método definido na interface criada.
```php
<?php  
  
declare(strict_types=1);  
  
namespace Tiagolopes\DesignPatternsPhp\Service\Taxes;  
  
use Tiagolopes\DesignPatternsPhp\Contracts\TaxTypeInterface;  
  
class ICMS implements TaxTypeInterface  
{  
    public function calculate(float $amount): float  
    {  
        return ($amount * 4) / 100;  
    }
}
```

Por fim, refatoramos a classe principal para que ela receba a classe do imposto e apenas chame o método `calculate` da classe específica.
```php
<?php  
  
declare(strict_types=1);  
  
namespace Tiagolopes\DesignPatternsPhp\Service;  
  
use Tiagolopes\DesignPatternsPhp\Contracts\TaxTypeInterface;  
  
class TaxCalculator  
{  
    private TaxTypeInterface $taxType;  
  
    public function calculate(float $amount): float  
    {  
        return $this->taxType->calculate($amount);  
    }  
    public function setTaxType(TaxTypeInterface $taxType): self  
    {  
        $this->taxType = $taxType;  
  
        return $this;  
    }
}
```

Também podemos organizar melhor a lógica no script que executa a classe utilizando um switch ou match para retornar a classe referente ao tipo de imposto utilizado nessa situação.
```php
<?php  
  
declare(strict_types=1);  
  
require_once __DIR__ . '/../vendor/autoload.php';  
  
use Tiagolopes\DesignPatternsPhp\Service\TaxCalculator;  
use Tiagolopes\DesignPatternsPhp\Service\Taxes\{ICMS, IPI, ISS};  
  
$taxCalculator = new TaxCalculator();  
  
$taxType = 'ISS';  
match ($taxType) {  
    'ICMS' => $taxCalculator->setTaxType(new ICMS()),  
    'IPI' => $taxCalculator->setTaxType(new IPI()),  
    'ISS' => $taxCalculator->setTaxType(new ISS()),  
    default => throw new InvalidArgumentException('Invalid taxType')  
};  
  
var_dump($taxCalculator->calculate(1000));
```

---
### Contéudos semelhantes

- Simple Factory: [[padrao-de-projetos-simple-factory]]
- Adapter: [[padrao-de-projetos-adapter]]
- State: [[padrao-de-projetos-state]]
- Facade: [[padrao-de-projetos-facade]]
- Observer: [[padrao-de-projetos-observer]]
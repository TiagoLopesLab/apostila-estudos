# Liskov Substitution Principle

> [!NOTE] Referências
> Vídeo: https://www.youtube.com/watch?v=f6-5ANuTkys
### Sobre

O _Liskov Substitution Principle_ nos diz que deve ser possível substituir uma classe por outra do mesmo tipo (filhas de uma mesma classe ou que implementam uma mesma interface), da mesma forma que deve ser possível substituir a classe filha pela classe pai sem quebrar o programa.

- **Pré-condição**: A subclasse não pode exigir mais do que a classe base exigia.
- **Pós condição**: A subclasse não pode reduzir as garantias fornecidas pela classe base após a execução do método.
- **Invariância**: A subclasse não pode alterar condições internas que a classe base mantinha constantes.
### Problemática 1

Considere que há uma classe que gera relatórios em CSV e implementa uma interface chamada `ReportGeneratorInterface` e em seu método `generate` retorna uma string contendo o caminho do arquivo gerado.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\Solid\LiskovSubstitution\Postcondition;

class CsvReportGenerator implements ReportGeneratorInterface
{
    private string $basePath;
    public function __construct()
    {
        $this->basePath = dirname(path: __DIR__, levels: 3) . '/reports';
    }

    public function generate(): string
    {
        // Lógica para geração do relatório
        $header = 'title;description';
        $body = 'Título;Descrição de exemplo';

        $filename = $this->basePath
            . '/'
            . uniqid(prefix: 'report_', more_entropy: true) . '.csv'
        ;
        file_put_contents(filename: $filename, data: $header . PHP_EOL . $body);

        return $filename;
    }
}
```

Há uma segunda classe que possui uma função parecida: gerar um relatório, mas dessa vez na AWS. Ela implementa a mesma interface da classe anterior e, no seu método `generate` retorna um link para o arquivo na AWS.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\Solid\LiskovSubstitution\Postcondition;

class S3ReportGenerator implements ReportGeneratorInterface
{
    private string $baseUrl;
    public function __construct()
    {
        $this->baseUrl = 'https://s3.amazonaws.com/mybucket';
    }

    public function generate(): string
    {
        // Lógica para geração do relatório

        $fileKey = uniqid(prefix: 's3_report', more_entropy: true) . '.txt';
        return "$this->baseUrl/$fileKey";
    }
}
```

Apesar de terem funções semelhantes (ambos geram um relatório), um dos arquivos é gerado localmente enquanto outro é gerado no ambiente de nuvem. Isso resulta em um problema exemplificado pelo código abaixo: se em uma determinada lógica for verificado se o arquivo existe localmente, dependendo do objeto utilizado a condição pode ser verdadeira ou falsa.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\Solid\LiskovSubstitution\Postcondition;

use DomainException;

class TestReportProcessor
{
    public function process(ReportGeneratorInterface $reportGenerator): void
    {
        $filepath = $reportGenerator->generate();

        if (!file_exists($filepath)) {
            throw new DomainException('O relatório não existe.');
        }

        echo 'Relatório processado' . PHP_EOL;
    }
}
```

Ao executar o código abaixo, usando uma instância da classe `CsvReportGenerator`, o código funciona corretamente pois a função `generate` está retornando o caminho de um arquivo local. Entretanto, ao chamar a mesma função, mas utilizando uma instância da classe `S3ReportGenerator`, é retornado o link para um arquivo em nuvem e, por consequência, acaba lançando uma exceção já que o arquivo não existe localmente. Isso se configura em um problema de **pós-condição**.
```php
<?php

declare(strict_types=1);

use Tiagolopes\Solid\LiskovSubstitution\Postcondition\CsvReportGenerator;
use Tiagolopes\Solid\LiskovSubstitution\Postcondition\S3ReportGenerator;
use Tiagolopes\Solid\LiskovSubstitution\Postcondition\TestReportProcessor;

require_once dirname(__DIR__) . '/vendor/autoload.php';

$CsvReportGenerator = new CsvReportGenerator();
$S3ReportGenerator = new S3ReportGenerator();
$reportProcessor = new TestReportProcessor();

$reportProcessor->process($CsvReportGenerator); // Funciona corretamente
$reportProcessor->process($S3ReportGenerator); // Lança a exceção
```
### Solução

Para esse caso específico, a solução mais adequada é criar interfaces diferentes para ambas as classes, por mais que elas possuem um objetivo em comum (gerar relatórios) e retornam o caminho do relatório, uma retorna um arquivo local enquanto outra retorna um arquivo em nuvem e isso já é o suficiente para se pensar em criar interfaces diferentes para as classes.

No exemplo abaixo, foi criada a interface `LocalReportGeneratorInterface` para ser implementada por classe que geram arquivos locais.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\Solid\LiskovSubstitution\Postcondition;

interface LocalReportGeneratorInterface
{
    public function generate(): string;
}
```

Também foi criada a classe `CloudReportGeneratorInterface`, que apesar de possuir o mesmo método que a interface anterior, é voltada para classes que geram relatórios armazenados em ambiente de nuvem.
```php
<?php

namespace Tiagolopes\Solid\LiskovSubstitution\Postcondition;

interface CloudReportGeneratorInterface
{
    public function generate(): string;
}
```

Dessa forma, ambas as classes passam a implementar uma interface diferente pois possuem um retorno com um objetivo levemente diferente, mas suficiente para impactar nas regras de negócio.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\Solid\LiskovSubstitution\Postcondition;

class S3LocalReportGenerator implements CloudReportGeneratorInterface
{
    private string $baseUrl;
    public function __construct()
    {
        $this->baseUrl = 'https://s3.amazonaws.com/mybucket';
    }

    public function generate(): string
    {
        // Lógica para geração do relatório

        $fileKey = uniqid(prefix: 's3_report', more_entropy: true) . '.txt';
        return "$this->baseUrl/$fileKey";
    }
}
```

Por fim, ao tentar utilizar uma instância de `S3ReportGenerator`, como no exemplo abaixo, é lançada uma exceção. Mas, não por conta da validação feita e sim por conta da incompatibilidade de tipos, já que agora ele espera somente classes que geram arquivos locais, ou seja, que implementam a interface `LocalReportGeneratorInterface`.
```php
<?php

declare(strict_types=1);

use Tiagolopes\Solid\LiskovSubstitution\Postcondition\CsvLocalReportGenerator;
use Tiagolopes\Solid\LiskovSubstitution\Postcondition\S3LocalReportGenerator;
use Tiagolopes\Solid\LiskovSubstitution\Postcondition\TestReportProcessor;

require_once dirname(__DIR__) . '/vendor/autoload.php';

$CsvReportGenerator = new CsvLocalReportGenerator();
$S3ReportGenerator = new S3LocalReportGenerator();
$reportProcessor = new TestReportProcessor();

$reportProcessor->process($CsvReportGenerator); // Funciona corretamente
// Lança uma exceção, dessa vez por serem de tipos diferentes
$reportProcessor->process($S3ReportGenerator);
```
---
### Problemática 2

No exemplo abaixo, temos uma classe chamada `BankAccount` que possui o método `deposit`. O método possui uma regra em que não é permitido passar um valor inferior a 0, ou seja, um número negativo pois só é possível fazer depósitos de valores positivos.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\Solid\LiskovSubstitution\Precondition;

use InvalidArgumentException;

class BankAccount
{
    public function __construct(
        protected(set) float $balance = 0
    ) {
    }

    public function deposit(float $amount): void
    {
        if ($amount <= 0) {
            throw new InvalidArgumentException(
	            'O valor do depósito deve ser maior que 0'
			);
        }

        $this->balance += $amount;
    }
}
```

Há uma segunda classe chamada `SavingsAccount` (uma Conta Poupança) que estende a classe `BankAccount`. Ela possui uma regra de depósitos acima de R$ 10 reais e, por isso, sobrescreve o método `deposit` adicionando uma nova validação. 
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\Solid\LiskovSubstitution\Precondition;

use InvalidArgumentException;

class SavingsAccount extends BankAccount
{
    public function deposit(float $amount): void
    {
        if ($amount < 10) {
            throw new InvalidArgumentException(
	            'O valor mínimo é de R$ 10,00 para a conta poupança'
			);
        }

        parent::deposit($amount);
    }
}
```

A classe do exemplo abaixo apenas serve para teste da classe `BankAccount` e as classes que a herdam.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\Solid\LiskovSubstitution\Precondition;

readonly class TestAccount
{
    public function __construct(
        private BankAccount $bankAccount
    ) {
    }

    public function deposit(float $amount): void
    {
        $this->bankAccount->deposit($amount);

        echo "O valor de R$ $amount foi depositado" . PHP_EOL;
    }
}
```

Ao testar ambas as classes de conta tentando fazer um depósito de R$ 5 reais, na instância de `BankAccount`, o depósito funciona corretamente, já na instância de `SavingsAccount` é retornado um erro pois ela possui uma validação a mais impedindo depósitos de menos de R$ 10 reais. Isso se configura um problema de **pré-condição** porque uma regra da classe filha adiciona uma regra a mais, que não possui na classe pai.
```php
<?php

declare(strict_types=1);

use Tiagolopes\Solid\LiskovSubstitution\Precondition\BankAccount;
use Tiagolopes\Solid\LiskovSubstitution\Precondition\SavingsAccount;
use Tiagolopes\Solid\LiskovSubstitution\Precondition\TestAccount;

require_once dirname(__DIR__) . '/vendor/autoload.php';

$bankAccount = new BankAccount();
$savingsAccount = new SavingsAccount();

$testAccount = new TestAccount($bankAccount);

$depositAmount = 5;
$testAccount->deposit($depositAmount); // Funciona corretamente

$testAccount = new TestAccount($savingsAccount);
$testAccount->deposit($depositAmount); // Lança uma exceção
```
### Solução

Nesse caso em específico, podemos considerar que a classe `BankAccount`, por já possuir uma regra de negócio atrelada que, a classe filha pode não compartilhar, não se trata de uma classe base, mas sim de uma outra classe filha.

Portanto, uma possível solução é tornar a classe base abstrata, para que não possa ser instanciada, e remover a regra de negócio que as filhas podem não compartilhar.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\Solid\LiskovSubstitution\Precondition;

use InvalidArgumentException;

abstract class BankAccount
{
    public function __construct(
        protected(set) float $balance = 0
    ) {
    }
}
```

A regra de não permitir depósito negativo passa a ser de uma outra classe filha, no exemplo abaixo ela foi chamada de `CheckingAccount` (Conta Poupança). Como a classe base não possui essa validação, ela não sobrescreve nenhuma regra existente.
```php
<?php

namespace Tiagolopes\Solid\LiskovSubstitution\Precondition;

use InvalidArgumentException;

class CheckingAccount extends BankAccount
{
    public function deposit(float $amount): void
    {
        if ($amount <= 0) {
            throw new InvalidArgumentException(
	            'O valor do depósito deve ser maior que 0'
			);
        }

        $this->balance += $amount;
    }
}
```

Isso, por si só, não evitaria o erro anterior, mas agora nenhuma das classes sobrescreve uma regra de um método da classe base. A solução apropriada sempre vai depender do contexto e da regra de negócios específica.

---
### Problemática 3

Considere mais uma vez um exemplo de conta bancária. No exemplo abaixo, a classe `BankAccount` possui um método de saque (ou `withdraw`). Essa função possui duas regras: o valor deve ser maior que 0 e menor ou igual ao saldo disponível.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\Solid\LiskovSubstitution\Invariance;

use DomainException;
use InvalidArgumentException;

class BankAccount
{
    public function __construct(
        protected(set) float $balance = 0
    ) {
    }

    public function withdraw(float $amount): void
    {
        if ($amount <= 0) {
            throw new InvalidArgumentException(
	            'O valor do saque deve ser maior que 0'
			);
        }

        if ($amount > $this->balance) {
            throw new DomainException(
	            'O valor do saque é maior que o saldo disponível'
			);
        }

        $this->balance -= $amount;
    }
}
```

Também há uma classe de Conta Poupança (`CheckingAccount`) que estende a classe `BankAccount`. Porém essa classe possui um novo atributo chamado `overdraftLimit` (limite adicional, como um cheque especial). Com isso, o método `withdraw` é sobrescrito e uma das regras dele é alterada, permitindo um saque maior que o saldo se ele possuir um limite adiciona que cobre essa despesa.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\Solid\LiskovSubstitution\Invariance;

use DomainException;
use InvalidArgumentException;

class CheckingAccount extends BankAccount
{
    public function __construct(
        float $balance = 0,
        protected(set) float $overdraftLimit = 0
    ) {
        parent::__construct($balance);
    }

    public function withdraw(float $amount): void
    {
        if ($amount <= 0) {
            throw new InvalidArgumentException(
	            'O valor do saque deve ser maior que 0'
			);
        }

        $availableBalance = $this->balance + $this->overdraftLimit;

        if ($amount > $availableBalance) {
            throw new DomainException(
	            'Saldo insuficiente e limite de cheque especial excedido.'
			);
        }

        $this->balance -= $amount;
    }
}
```

Novamente, para complementar o exemplo, foi criada uma classe de testes que recebe uma instância de `BankAccount` ou de qualquer classe que a herda.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\Solid\LiskovSubstitution\Invariance;

readonly class TestAccount
{
    public function __construct(
        private BankAccount $bankAccount
    ) {
    }

    public function withdraw(float $amount): void
    {
        $this->bankAccount->withdraw($amount);

        echo "Saque de R$ $amount realizado" . PHP_EOL;
    }
}
```

No exemplo abaixo, ao fazer o teste de saque de 200 reais em cada tipo de conta (tendo 150 de saldo disponível), a instância de `CheckingAccount` executa o método sem erro, já que possui a regra de limite adicional, enquanto a instância da classe base retorna um erro pois não possui essa regra. Isso se configura em um problema de **invariância** já que a classe filha modifica uma regra da classe base.
```php
<?php

declare(strict_types=1);

use Tiagolopes\Solid\LiskovSubstitution\Invariance\BankAccount;
use Tiagolopes\Solid\LiskovSubstitution\Invariance\CheckingAccount;
use Tiagolopes\Solid\LiskovSubstitution\Invariance\TestAccount;

require_once dirname(__DIR__) . '/vendor/autoload.php';

$bankAccount = new BankAccount(balance: 150);
$checkingAccount = new CheckingAccount(balance: 150, overdraftLimit: 100);

$testAccount = new TestAccount($checkingAccount);
$testAccount->withdraw(200); // Realiza o saque por conta do limite adicional

$testAccount = new TestAccount($bankAccount);
$testAccount->withdraw(200); // Lança uma exceção
```
### Solução

Novamente, não existe uma solução padrão. Depende do contexto e das regras de negócio. Para esse exemplo específico, poderia ser implementada a mesma solução do exemplo anterior, deixando a classe base sem nenhuma regra de saque (ou apenas a regra de ser um valor superior a 0, que, em tese, seria comum em todas as classes). Dessa forma, não haveria uma sobrescrita de uma regra da classe pai.
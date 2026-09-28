# Padrão de Projetos Singleton

> [!NOTE] Vídeo
> https://www.youtube.com/watch?v=E8ey3HjSthg
### Sobre

### Problemática

```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatterns\Singleton;

use RuntimeException;

class Logger
{
    public function log(string $filename, string $content): void
    {
        $resource = fopen(filename: $filename, mode: 'a');

        if (!$resource) {
            throw new RuntimeException('Não foi possível abrir o arquivo.');
        }

        $date = date('Y-m-d H:i:s');
        $result = fwrite(
            stream: $resource,
            data: "[$date] - $content" . PHP_EOL
        );

        if ($result === false) {
            throw new RuntimeException('Não foi possível escrever no arquivo.');
        }

        fclose($resource);
    }
}
```

```php
<?php

declare(strict_types=1);

date_default_timezone_set('America/Sao_Paulo');

require_once dirname(__DIR__) . '/vendor/autoload.php';

use Tiagolopes\DesignPatterns\Singleton\Logger;

$logger = new Logger();
$logger->log(filename: 'app.log', content: 'Conteúdo do log');

$logger2 = new Logger();

var_dump($logger); // Instância 1
var_dump($logger2); // Instância 2 (referências diferentes)
```
### Solução

```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatterns\Singleton;

use RuntimeException;

class Logger
{
    private string $filename;
    private static ?self $instance = null;

    private function __construct()
    {
        $this->filename = dirname(path: __DIR__, levels: 2) . '/app.log';
    }

    public static function getInstance(): self
    {
        if (self::$instance === null) {
            self::$instance = new self();
        }

        return self::$instance;
    }

    public function log(string $content): void
    {
        $resource = fopen(filename: $this->filename, mode: 'a');

        if (!$resource) {
            throw new RuntimeException('Não foi possível abrir o arquivo.');
        }

        $date = date('Y-m-d H:i:s');
        $result = fwrite(
            stream: $resource,
            data: "[$date] - $content" . PHP_EOL
        );

        if ($result === false) {
            throw new RuntimeException('Não foi possível escrever no arquivo.');
        }

        fclose($resource);
    }
}
```

```php
<?php

declare(strict_types=1);

date_default_timezone_set('America/Sao_Paulo');

require_once dirname(__DIR__) . '/vendor/autoload.php';

use Tiagolopes\DesignPatterns\Singleton\Logger;

$logger = Logger::getInstance();
$logger->log(content: 'Conteúdo do log');

$logger2 = Logger::getInstance();

var_dump($logger);
var_dump($logger2); // Mesma referência
```
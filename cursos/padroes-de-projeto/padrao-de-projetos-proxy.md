# Padrão de Projetos Proxy

> [!NOTE] Vídeo
> https://www.youtube.com/watch?v=el1MtIPXTqo

### Sobre

### Problemática

```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatterns\Proxy;

use RuntimeException;

class ReportGenerator
{
    private(set) string $path;

    public function __construct()
    {
        $this->path = dirname(__DIR__, 2) . '/reports';
    }

    public function generate(Report $report): string
    {
        // Lógica para montagem do relatório

        sleep(seconds: 5); // Simulando uma requisição demorada

        $filename = "$this->path/report_$report->id.txt";
        $result = file_put_contents(
            filename: $filename,
            data: $report->content
        );

        if ($result === false) {
            throw new RuntimeException('Não foi possível gerar o relatório');
        }

        return $filename;
    }
}
```

```php
<?php

declare(strict_types=1);

require_once dirname(__DIR__) . '/vendor/autoload.php';

use Tiagolopes\DesignPatterns\Proxy\ReportGenerator;
use Tiagolopes\DesignPatterns\Proxy\ReportRepository;

if (count($argv) < 2) {
    throw new DomainException('Script deve receber um segundo argumento');
}

$reportId = (int) $argv[1];

$report = new ReportRepository()->findById($reportId);

// Demora 5 segundos para entregar cada relatório
$filename = new ReportGenerator()->generate($report);
echo $filename . PHP_EOL;
```
### Solução

```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatterns\Proxy;

interface ReportGeneratorInterface
{
    public function generate(Report $report): string;
}
```

```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatterns\Proxy;

readonly class ReportGeneratorCacheProxy implements ReportGeneratorInterface
{
    public function __construct(
        private ReportGenerator $reportGenerator
    ) {
    }

    public function generate(Report $report): string
    {
        $filename = "{$this->reportGenerator->path}/report_$report->id.txt";
        if (file_exists($filename)) {
            return $filename;
        }

        return $this->reportGenerator->generate($report);
    }
}
```

```php
<?php

declare(strict_types=1);

require_once dirname(__DIR__) . '/vendor/autoload.php';

use Tiagolopes\DesignPatterns\Proxy\ReportGenerator;
use Tiagolopes\DesignPatterns\Proxy\ReportGeneratorCacheProxy;
use Tiagolopes\DesignPatterns\Proxy\ReportRepository;

if (count($argv) < 2) {
    throw new DomainException('Script deve receber um segundo argumento');
}

$reportId = (int) $argv[1];

$report = new ReportRepository()->findById($reportId);
$reportGenerator = new ReportGenerator();
$reportGeneratorCache = new ReportGeneratorCacheProxy($reportGenerator);

// Retorna instantaneamente se for um relatório já gerado
$filename = $reportGeneratorCache->generate($report);
echo $filename . PHP_EOL;
```
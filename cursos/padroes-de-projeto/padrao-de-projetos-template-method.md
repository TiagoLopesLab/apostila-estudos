# Padrão de Projetos Template Method

> [!NOTE] Vídeo
> https://youtu.be/j5fGTi8ObK4?si=yyHuvNppgFsiCcSw
### Sobre

O _Template Method_ é um Padrão de Projeto comportamental que visa definir um "esqueleto" de uma determinada lógica na superclasse (ou classe base) e reutilizar esse "esqueleto" nas classes filhas que realizam as mesmas etapas de forma diferente, permitindo que essas classes sobrescrevam os códigos necessários mantendo a lógica base da classe pai.
### Problemática

No exemplo abaixo, temos uma classe que realiza uma mineração de dados em um arquivo do tipo CSV, realizando várias etapas, desde a leitura do arquivo até o envio do relatório. A princípio, não há nenhum problema de arquitetura nessa classe.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatternsPhp\DataMiner;

class CsvDataMiner
{
    public function mine(string $path): void
    {
        $fileContent = $this->openFile($path);
        $rawData = $this->extractCsvData($fileContent);
        $parsedData = $this->parseCsvData($rawData);
        $report = $this->analyseData($parsedData);
        $this->sendReport($report);
    }

    private function openFile(string $path): string
    {
        return "File content of $path";
    }

    private function extractCsvData(string $fileContent): array
    {
        return ['raw_data_csv' => $fileContent];
    }

    private function parseCsvData(array $rawData): array
    {
        return ['parsed_data_csv' => $rawData];
    }

    private function analyseData(array $parsedData): array
    {
        return ['analysed_data' => $parsedData];
    }

    private function sendReport(array $analysedData): void
    {
        echo 'Enviando o relatório';
    }
}
```

Mas, imagine que foi solicitado que seja possível realizar a mineração de dados em um arquivo do tipo DOC e, por ser um arquivo diferente, a forma de processá-lo para coletar os dados também seria diferente. Portanto, foi necessário criar uma outra classe que realiza as mesmas etapas, porém com algumas das etapas com uma lógica interna diferente. 
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatternsPhp\DataMiner;

class DocDataMiner
{
    public function mine(string $path): void
    {
        $fileContent = $this->openFile($path);
        $rawData = $this->extractDocData($fileContent);
        $parsedData = $this->parseDocData($rawData);
        $report = $this->analyseData($parsedData);
        $this->sendReport($report);
    }

    private function openFile(string $path): string
    {
        return "File content of $path";
    }

    private function extractDocData(string $fileContent): array
    {
        return ['raw_data_doc' => $fileContent];
    }

    private function parseDocData(array $rawData): array
    {
        return ['parsed_data_doc' => $rawData];
    }

    private function analyseData(array $parsedData): array
    {
        return ['analysed_data' => $parsedData];
    }

    private function sendReport(array $analysedData): void
    {
        echo 'Enviando o relatório';
    }
}
```

O problema disso é que, se for solicitado que seja possível fazer a mineração de dados em um arquivo PDF teria que criar uma outra classe com exatamente as mesmas etapas, mas uma implementação interna diferente.
### Solução

Para resolvermos isso, podemos aplicar o padrão _Template Method_. Para isso, primeiro temos que criar uma classe abstrata que define os métodos de todas as etapas necessária e implementa somente aqueles que são comuns em todas as classes (ou pelo menos na maioria delas). O restante das funções é definido como `abstract` para que a classe filha implemente a lógica de acordo com o processamento necessário.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatternsPhp\DataMiner;

abstract class DataMiner
{
    public function mine(string $path): void
    {
        $fileContent = $this->openFile($path);
        $rawData = $this->extractData($fileContent);
        $parsedData = $this->parseData($rawData);
        $report = $this->analyseData($parsedData);
        $this->sendReport($report);
    }

    protected abstract function openFile(string $path): string;
    protected abstract function extractData(string $fileContent): array;
    protected abstract function parseData(array $rawData): array;
    protected function analyseData(array $parsedData): array
    {
        return ['analysed_data' => $parsedData];
    }

    protected function sendReport(array $analysedData): void
    {
        echo 'Enviando o relatório' . PHP_EOL;
    }
}
```

Dessa forma, basta que a classe filha estenda a classe base e implemente os métodos abstratos.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatternsPhp\DataMiner;

class CsvDataMiner extends DataMiner
{
    protected function openFile(string $path): string
    {
        return "File content of $path";
    }

    protected function extractData(string $fileContent): array
    {
        return ['raw_data_csv' => $fileContent];
    }

    protected function parseData(array $rawData): array
    {
        return ['parsed_data_csv' => $rawData];
    }
}
```

Assim, se for necessário implementar uma nova forma de mineração (por PDF, por exemplo), basta criar uma classe que estende da classe base e implemente os métodos necessários sem ter que repetir o mesmo código, já que o que é comum a todos fica na classe base.

---
- Simple Factory: [[padrao-de-projetos-simple-factory]]
- Strategy: [[padrao-de-projetos-strategy]]
- State: [[padrao-de-projetos-state]]
- Adapter: [[padrao-de-projetos-adapter]]
- Facade: [[padrao-de-projetos-facade]]
- Observer: [[padrao-de-projetos-observer]]

# Padrão de Projetos Adapter

> [!NOTE] Link do vídeo
> https://youtu.be/Fg1kEjaaBrs?si=xPZB9xc1qenvJx55

O padrão Adapter consiste em criar classes que realizam a adaptação de uma biblioteca ou código externo para que uma lógica interna consiga utilizar sem precisar saber como esse código externo funciona.
### Problemática

No exemplo abaixo, temos a implementação de uma lógica de geração de relatórios em PDF utilizando especificamente o pacote Dompdf. O problema nesse caso é que se o time de desenvolvimento precisar trocar de pacote ou, então, atualizar a lógica de impressão por conta de uma atualização da biblioteca será necessário alterar diretamente na classe `SalesReportGenerator` ferindo um dos princípios do SOLID.
```php
<?php  
  
declare(strict_types=1);  
  
namespace Tiagolopes\DesignPatternsPhp\Service;  
  
use DateTime;  
use Dompdf\Dompdf;  
  
class SalesReportGenerator  
{  
    public function generate(): void  
    {  
        $domPdf = new DomPdf();  
        $domPdf->loadHtml('Conteúdo do relatório.');  
        $domPdf->setPaper('A4', 'landscape');  
        $domPdf->render();  
  
        $filename = new DateTime()->getTimestamp() . '.pdf';  
  
        file_put_contents(
	        dirname(__DIR__, 2) . "/report/$filename",
	        $domPdf->output()
		);  
    }
}
```

### Solução

Podemos corrigir isso utilizando o padrão Adapter. Primeiramente, é necessário criar uma interface pela qual todas as classes adaptadores irão estender, fornecendo um método comum a todas elas.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatternsPhp\Contracts;

interface PdfAdapterInterface
{
    public function generate(
	    string $fileName,
	    string $content,
	    array $params = []
    ): void;
}
```

Após isso, criamos uma classe adaptadora específica para a biblioteca Dompdf. Dessa forma, sempre que o pacote for atualizado e necessitar de uma alteração na implementação, será alterado nessa classe e não no service `SalesReportGenerator` que é responsável pelas regras de negócio.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatternsPhp\Service\Adapters;

use Dompdf\Dompdf;
use Tiagolopes\DesignPatternsPhp\Contracts\PdfAdapterInterface;

class DomPdfAdapter implements PdfAdapterInterface
{
    public function generate(
	    string $fileName,
	    string $content,
	    array $params = []
	): void
    {
        $domPdf = new DomPdf();
        $domPdf->loadHtml($content);
        $domPdf->setPaper(
            size: $params['size'] ?? 'A4',
            orientation: $params['orientation'] ?? 'landscape'
        );
        $domPdf->render();

        file_put_contents(
            filename: dirname(path: __DIR__, levels: 3) . "/report/$fileName",
            data: $domPdf->output()
        );
    }
}
```

Se precisarmos utilizar uma outra biblioteca, como a Mpdf por exemplo, basta criarmos uma classe adaptadora para ela estendendo a mesma interface.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatternsPhp\Service\Adapters;

use Mpdf\Mpdf;
use Mpdf\MpdfException;
use Tiagolopes\DesignPatternsPhp\Contracts\PdfAdapterInterface;

class MpdfAdapter implements PdfAdapterInterface
{
    public function generate(
	    string $fileName,
	    string $content,
	    array $params = []
	): void
    {
        $mpdf = new Mpdf();
        $mpdf->WriteHTML($content);

        file_put_contents(
            filename: dirname(path: __DIR__, levels: 3) . "/report/$fileName",
            data: $mpdf->OutputBinaryData()
        );
    }
}
```

---
### Conteúdos semelhantes

- Simple Factory: [[padrao-de-projetos-simple-factory]]
- Strategy: [[padrao-de-projetos-strategy]]
- State: [[padrao-de-projetos-state]]
- Facade: [[padrao-de-projetos-facade]]
- Observer: [[padrao-de-projetos-observer]]
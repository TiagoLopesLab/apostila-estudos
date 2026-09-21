# Padrão de projetos Simple Factory

Vídeo: https://www.youtube.com/watch?v=3-ESljj0jgI

> [!WARNING] Importante
> Esse padrão não é oficial do GOF (Gang of Four).

O padrão **Simple Factory** é um padrão de projetos simples que consiste em criar uma classe fabricadora de objetos diferentes, mas que implementam uma mesma interface.
### Problemática

Considere o código abaixo. A classe `NotificationService` tem como objetivo enviar uma notificação por um canal específico. Para saber qual é o tipo de notificação, o método `sendNotification` faz vários `if's`, fazendo com que, a cada novo tipo de notificação seja necessário alterar a classe. Isso fere um dos princípios do SOLID, pois a classe possui mais de um motivo para mudar, além de enviar as notificações ela também tem que verificar o tipo de notificação.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatternsPhp\Service;

use InvalidArgumentException;
use Tiagolopes\DesignPatternsPhp\Service\Notification\EmailNotification;
use Tiagolopes\DesignPatternsPhp\Service\Notification\SMSNotification;
use Tiagolopes\DesignPatternsPhp\Service\Notification\WhatsappNotification;

class NotificationService
{
    public function sendNotification(
	    string $notificationType,
	    string $message
	): string
    {
        if ($notificationType === 'sms') {
            $notification = new SMSNotification();
        } elseif ($notificationType === 'email') {
            $notification = new EmailNotification();
        } elseif ($notificationType === 'whatsapp') {
            $notification = new WhatsappNotification();
        } else {
            throw new InvalidArgumentException('Notification type not supported');
        }

        return $notification->send($message);
    }
}
```

### Solução

O padrão **Simple Factory** surge para resolver esse problema. Para isso, nós criamos uma classe irá abstrair toda a lógica de verificação do tipo de notificação e irá retornar a classe específica de acordo com o tipo.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatternsPhp\Factories;

use InvalidArgumentException;
use Tiagolopes\DesignPatternsPhp\Contracts\NotificationInterface;
use Tiagolopes\DesignPatternsPhp\Service\Notification\EmailNotification;
use Tiagolopes\DesignPatternsPhp\Service\Notification\SMSNotification;
use Tiagolopes\DesignPatternsPhp\Service\Notification\WhatsappNotification;

class NotificationFactory
{
    public static function create(string $notificationType): NotificationInterface
    {
        return match ($notificationType) {
            'sms' => new SMSNotification(),
            'email' => new EmailNotification(),
            'whatsapp' => new WhatsappNotification(),
            default => throw new InvalidArgumentException('Invalid option.')
        };
    }
}
```

Dessa forma, o método `sendNotification` apenas utiliza a classe factory para gerar o objeto correto de acordo com o tipo e envia a notificação, passando a ter apenas um único motivo para mudar.
```php
<?php

declare(strict_types=1);

namespace Tiagolopes\DesignPatternsPhp\Service;

use Tiagolopes\DesignPatternsPhp\Factories\NotificationFactory;

class NotificationService
{
    public function sendNotification(
	    string $notificationType,
	    string $message
	): string
    {
        $notification = NotificationFactory::create($notificationType);
        return $notification->send($message);
    }
}
```

Assim, se for necessário incluir um novo tipo de notificação, apenas a classe fabricadora vai ser modificada, enquanto o service não precisa saber dessa mudança e nem quais formas de notificação existem.

---
### Conteúdos semelhantes

- Strategy: [[padrao-de-projetos-strategy]]
- Adapter: [[padrao-de-projetos-adapter]]
- State: [[padrao-de-projetos-state]]
- Facade: [[padrao-de-projetos-facade]]
- Observer: [[padrao-de-projetos-observer]]
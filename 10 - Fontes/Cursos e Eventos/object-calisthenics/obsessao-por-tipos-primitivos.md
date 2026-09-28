# Obsessão por tipos primitivos

> [!NOTE] Referências
> Vídeo: https://www.youtube.com/watch?v=PXHqooNlkVM

```php
<?php

namespace Tiagolopes\ObjectCalisthenics\PrimitiveObsession;

use DomainException;

readonly class Customer
{
    public string $email;
    public string $cpf;
    private function __construct(string $email, string $cpf)
    {
        $this->validateEmail($email);
        $formattedCpf = $this->validateCpf($cpf);

        $this->email = $email;
        $this->cpf = $formattedCpf;
    }

    private function validateEmail(string $email): void
    {
        $validatedEmail = filter_var($email, FILTER_VALIDATE_EMAIL);

        if ($validatedEmail === false) {
            throw new DomainException('Invalid Email');
        }
    }

    private function validateCpf(string $cpf): string
    {
        $formattedCpf = preg_replace(
	        pattern: '/[^0-9]/i',
	        replacement: '',
	        subject: $cpf
		);

        if (is_null($formattedCpf) || strlen($formattedCpf) !== 11) {
            throw new DomainException('Invalid CPF');
        }

        return $formattedCpf;
    }
}
```

```php
<?php

namespace Tiagolopes\ObjectCalisthenics\PrimitiveObsession;

use DomainException;

readonly class Employee
{
    public string $email;
    public string $cpf;
    private function __construct(string $email, string $cpf)
    {
        $this->validateEmail($email);
        $formattedCpf = $this->validateCpf($cpf);

        $this->email = $email;
        $this->cpf = $formattedCpf;
    }

    private function validateEmail(string $email): void
    {
        $validatedEmail = filter_var($email, FILTER_VALIDATE_EMAIL);

        if ($validatedEmail === false) {
            throw new DomainException('Invalid Email');
        }
    }

    private function validateCpf(string $cpf): string
    {
        $formattedCpf = preg_replace(
	        pattern: '/[^0-9]/i',
	        replacement: '',
	        subject: $cpf
		);

        if (is_null($formattedCpf) || strlen($formattedCpf) !== 11) {
            throw new DomainException('Invalid CPF');
        }

        return $formattedCpf;
    }
}
```

```php
<?php

namespace Tiagolopes\ObjectCalisthenics\PrimitiveObsession\ValueObjects;

use DomainException;

readonly class Email
{
    private function __construct(
        private string $email
    ) {
    }

    public static function fromString(string $email): self
    {
        self::validateEmail($email);

        return new self($email);
    }

    private static function validateEmail(string $email): void
    {
        $validatedEmail = filter_var($email, FILTER_VALIDATE_EMAIL);

        if ($validatedEmail === false) {
            throw new DomainException('Invalid Email');
        }
    }

    public function toValue(): string
    {
        return $this->email;
    }
}
```

```php
<?php

namespace Tiagolopes\ObjectCalisthenics\PrimitiveObsession\ValueObjects;

use DomainException;

readonly class Cpf
{
    private function __construct(
        private string $cpf
    ) {
    }

    public static function fromString(string $cpf): self
    {
        $validatedCpf = self::validateCpf($cpf);

        return new self($validatedCpf);
    }

    private static function validateCpf(string $cpf): string
    {
        $formattedCpf = preg_replace(
	        pattern: '/[^0-9]/i',
	        replacement: '',
	        subject: $cpf
		);

        if (is_null($formattedCpf) || strlen($formattedCpf) !== 11) {
            throw new DomainException('Invalid CPF');
        }

        return $formattedCpf;
    }

    public function toValue(): string
    {
        return $this->cpf;
    }
}
```

```php
<?php

namespace Tiagolopes\ObjectCalisthenics\PrimitiveObsession;

use Tiagolopes\ObjectCalisthenics\PrimitiveObsession\ValueObjects\Cpf;
use Tiagolopes\ObjectCalisthenics\PrimitiveObsession\ValueObjects\Email;

readonly class Customer
{
    private function __construct(
        private Email $email,
        private Cpf $cpf
    ) {
    }

    public function email(): string
    {
        return $this->email->toValue();
    }

    public function cpf(): string
    {
        return $this->cpf->toValue();
    }
}
```

```php
<?php

namespace Tiagolopes\ObjectCalisthenics\PrimitiveObsession;

use Tiagolopes\ObjectCalisthenics\PrimitiveObsession\ValueObjects\Cpf;
use Tiagolopes\ObjectCalisthenics\PrimitiveObsession\ValueObjects\Email;

readonly class Employee
{
    private function __construct(
        private Email $email,
        private Cpf $cpf
    ) {
    }

    public function email(): string
    {
        return $this->email->toValue();
    }

    public function cpf(): string
    {
        return $this->cpf->toValue();
    }
}
```
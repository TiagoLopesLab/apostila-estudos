# Classes enxutas e com poucas variáveis

```php
<?php

namespace Tiagolopes\ObjectCalisthenics\SmallClasses;

readonly class Customer
{
    public function __construct(
        private string $name,
        private string $email,
        private string $phoneNumber,
        private string $street,
        private string $zipCode,
        private string $number,
        private string $neighboor,
        private string $city,
        private string $state
    ) {
    }

    public function getName(): string
    {
        return $this->name;
    }

    public function getEmail(): string
    {
        return $this->email;
    }

    public function getPhoneNumber(): string
    {
        return $this->phoneNumber;
    }

    public function getStreet(): string
    {
        return $this->street;
    }

    public function getNumber(): string
    {
        return $this->number;
    }

    // Other getters
}
```

```php
<?php

namespace Tiagolopes\ObjectCalisthenics\SmallClasses;

readonly class Address
{
    public function __construct(
        public string $street,
        public string $zipCode,
        public string $number,
        public string $neighbor,
        public string $city,
        public string $state
    ) {
    }
}
```

```php
<?php

namespace Tiagolopes\ObjectCalisthenics\SmallClasses\ValueObjects;

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

namespace Tiagolopes\ObjectCalisthenics\SmallClasses\ValueObjects;

use DomainException;

readonly class Phone
{
    private function __construct(
        private string $phoneNumber
    ) {
    }

    public static function fromString(string $phoneNumber): self
    {
        self::validateEmail($phoneNumber);

        return new self($phoneNumber);
    }

    private static function validateEmail(string $phoneNumber): void
    {
        $formattedPhoneNumber = preg_replace(
	        pattern: '/\D/',
	        replacement: '',
	        subject: $phoneNumber
		);

        $regex = '/^[1-9]{2}9[0-9]{8}$/';
        $isValid = preg_match($regex, $formattedPhoneNumber);

        if ($isValid === false) {
            throw new DomainException('Invalid Phone number');
        }
    }

    public function toValue(): string
    {
        return $this->phoneNumber;
    }
}
```

```php
<?php

namespace Tiagolopes\ObjectCalisthenics\SmallClasses;

use Tiagolopes\ObjectCalisthenics\SmallClasses\ValueObjects\Email;
use Tiagolopes\ObjectCalisthenics\SmallClasses\ValueObjects\Phone;

readonly class ContactInfo
{
    public function __construct(
        private Email $email,
        private Phone $phone
    ) {
    }

    public function getEmail(): string
    {
        return $this->email->toValue();
    }

    public function getPhone(): string
    {
        return $this->phone->toValue();
    }
}
```

```php
<?php

namespace Tiagolopes\ObjectCalisthenics\SmallClasses;

readonly class Customer
{
    public function __construct(
        public string $name,
        private ContactInfo $contactInfo,
        public Address $address
    ) {
    }

    public function getEmail(): string
    {
        return $this->contactInfo->getEmail();
    }

    public function getPhone(): string
    {
        return $this->contactInfo->getPhone();
    }
}
```
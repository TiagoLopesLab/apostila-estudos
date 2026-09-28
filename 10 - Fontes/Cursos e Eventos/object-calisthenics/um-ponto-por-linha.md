# Use apenas um ponto por linha

> [!NOTE] Referências
> Vídeo: https://www.youtube.com/watch?v=KXaPJhG9yCk

```php
<?php

namespace Tiagolopes\ObjectCalisthenics\OneDotPerLine;

class ShippingCalculator
{
    public function calculate(Order $order): float
    {
        $zipCode = $order->getCustomer()->getAddress()->zipCode;

        // Lógica para calcular o frete
        return rand(10, 500) / 10;
    }
}
```

```php
<?php

namespace Tiagolopes\ObjectCalisthenics\OneDotPerLine;

readonly class Order
{
    public function __construct(
        public float $amount,
        public string $product,
        private Customer $customer
    ) {
    }

    public function getCustomer(): Customer
    {
        return $this->customer;
    }
}
```

```php
<?php

namespace Tiagolopes\ObjectCalisthenics\OneDotPerLine;

readonly class Customer
{
    public function __construct(
        private Address $address
    ) {
    }

    public function getAddress(): Address
    {
        return $this->address;
    }
}
```

```php
<?php

namespace Tiagolopes\ObjectCalisthenics\OneDotPerLine;

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
## Solução

```php
<?php

namespace Tiagolopes\ObjectCalisthenics\OneDotPerLine;

readonly class Customer
{
    public function __construct(
        private Address $address
    ) {
    }

    public function getZipCode(): string
    {
        return $this->address->zipCode;
    }
}
```

```php
<?php

namespace Tiagolopes\ObjectCalisthenics\OneDotPerLine;

readonly class Order
{
    public function __construct(
        public float $amount,
        public string $product,
        private Customer $customer
    ) {
    }

    public function getCustomerZipCode(): string
    {
        return $this->customer->getZipCode();
    }
}
```

```php
<?php  
  
namespace Tiagolopes\ObjectCalisthenics\OneDotPerLine;  
  
class ShippingCalculator  
{  
    public function calculate(Order $order): float  
    {  
        $zipCode = $order->getCustomerZipCode();  
  
        // Lógica para calcular o frete  
        return rand(10, 500) / 10;  
    }
}
```
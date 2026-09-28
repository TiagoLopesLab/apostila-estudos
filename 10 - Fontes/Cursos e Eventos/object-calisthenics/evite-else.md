# Evite usar o operador Else

```php
<?php

namespace Tiagolopes\ObjectCalisthenics\DontUseElse;

use DateTime;
use DomainException;

class Report
{
    private const string REPORT_FOLDER = '/reports';

    public function generate(string $key, string $content): string
    {
        if (strlen(trim($content)) > 0) {
            $path = dirname(path: __DIR__, levels: 2) . self::REPORT_FOLDER;
            if (file_exists($path) && is_dir($path)) {
                $filename = 
	                $path . "/{$key}_" . new DateTime()->getTimestamp() . '.txt';
                $resource = fopen($filename, 'a');

                if ($resource !== false) {
                    $result = fputs($resource, $content);
                    fclose($resource);
                    if ($result !== false) {
                        return $filename;
                    } else {
                        throw new DomainException('Failed to update file!');
                    }
                } else {
                    throw new DomainException('Report not generated!');
                }
            } else {
                throw new DomainException('Folder "reports" does not exist!');
            }
        } else {
            throw new DomainException('Content cannot be empty');
        }
    }

    public function findLastReport(string $key): ?string
    {
        $path = dirname(path: __DIR__, levels: 2) . self::REPORT_FOLDER;
        $files = scandir($path) ?: [];

        $matchedFiles = array_filter(
	        $files,
	        fn (string $file) => str_starts_with($file, $key)
		);

        if (count($matchedFiles) > 0) {
            $file = $path . '/' . array_pop($matchedFiles);
        } else {
            $file = null;
        }

        return $file;
    }
}
```

```php
public function generate(string $key, string $content): string
{
	if (strlen(trim($content)) === 0) {
		throw new DomainException('Content cannot be empty');
	}

	$path = dirname(path: __DIR__, levels: 2) . self::REPORT_FOLDER;
	if (!file_exists($path) || !is_dir($path)) {
		throw new DomainException('Folder "reports" does not exist!');
	}

	$filename = $path . "/{$key}_" . new DateTime()->getTimestamp() . '.txt';
	$resource = fopen($filename, 'a');
	if ($resource === false) {
		throw new DomainException('Report not generated!');
	}

	$result = fputs($resource, $content);
	fclose($resource);
	if ($result === false) {
		throw new DomainException('Failed to update file!');
	}

	return $filename;
}
```

```php
public function findLastReport(string $key): ?string
{
	$path = dirname(path: __DIR__, levels: 2) . self::REPORT_FOLDER;
	$files = scandir($path) ?: [];

	$matchedFiles = array_filter(
		$files,
		fn (string $file) => str_starts_with($file, $key)
	);

	if (count($matchedFiles) === 0) {
		return null;
	}

	return $path . '/' . array_pop($matchedFiles);
}
```
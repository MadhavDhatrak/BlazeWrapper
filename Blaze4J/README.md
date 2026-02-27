**Blaze4J** is a high-performance Java wrapper for the Blaze JSON Schema validator, enabling seamless integration between Java applications and the native C++ Blaze engine. By leveraging Java’s Foreign Function & Memory (FFM) API, Blaze4J delivers near-native performance while preserving memory safety.

Blaze4J enables Java developers to validate JSON documents against JSON Schema specifications (supporting multiple draft versions) with minimal overhead and maximum efficiency.

---

<p align="center">
  <img src="https://github.com/user-attachments/assets/5b1f7e38-4614-446e-985c-6ad2cc135ff3" width="516" height="346" alt="Blaze4J Architecture" />
</p>

## Table of Contents

* [Overview](#overview)
* [Features](#features)
* [Installation](#installation)
* [Usage](#usage)
* [Supported Drafts](#supported-drafts)
* [Supported Resolvers](#supported-resolvers)
* [API Documentation](#api-documentation)

  * [Compiler API](./docs/Compiler.md)
  * [Validator API](./docs/Validator.md)
* [Best Practices](#best-practices)
* [Contributing](#contributing)
* [Credits](#credits)

---

## Overview

Blaze4J bridges Java and native C++ for **fast, standards-compliant JSON Schema validation**. It supports multiple schema drafts, efficient memory management via the FFM API, and robust error reporting.

---

## Features

* **High-performance**: Native C++ validation with minimal Java overhead.
* **Multi-draft support**: Compatible with JSON Schema Drafts 4, 6, 7, 2019-09, and 2020-12.
* **Memory safety**: Uses Java's Foreign Function & Memory (FFM) API for safe, efficient resource management.
* **Detailed error reporting**: Get precise validation errors and paths.
* **Easy integration**: Simple API for compiling schemas and validating instances.
* **Cross-platform**: Works on Windows, Linux, and macOS.
* **Zero-copy validation path** for minimal overhead.

---

## Installation

### Maven

Add the following dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>io.github.madhavdhatrak</groupId>
    <artifactId>blaze4j</artifactId>
    <version>0.0.3</version>
</dependency>
```

### Gradle

Add this to your `build.gradle`:

```gradle
implementation group: 'io.github.madhavdhatrak', name: 'blaze4j', version: '0.0.3'
```

---

## Running Tests

If using Maven:

```bash
mvn test
```

To run a specific main class (requires exec plugin):

```bash
mvn compile exec:java -Dexec.mainClass=com.example.Test
```

*Replace `com.example.Test` with your class name.*

---

> [!IMPORTANT]
> For more details, check out the repository:
> **https://github.com/MadhavDhatrak/Blaze4J-examples.git**
> It contains examples of both Spring Boot and plain Java applications to help you set up the library quickly.

---

## Usage

### Simple Boolean Result

```java
package com.example;

import com.github.madhavdhatrak.blaze4j.BlazeValidator;
import com.github.madhavdhatrak.blaze4j.CompiledSchema;
import com.github.madhavdhatrak.blaze4j.SchemaCompiler;

public class Test {
    public static void main(String[] args) {
        String schemaJson = """
            {
                "$schema": "https://json-schema.org/draft/2020-12/schema",
                "type": "string"
            }
            """;

        SchemaCompiler compiler = new SchemaCompiler();
        try (CompiledSchema schema = compiler.compile(schemaJson)) {
            BlazeValidator validator = new BlazeValidator();

            boolean validStringResult = validator.validate(schema, "\"hello\"");
            System.out.println("Validation result for \"hello\": " + validStringResult);

            boolean invalidNumberResult = validator.validate(schema, "42");
            System.out.println("Validation result for 42: " + invalidNumberResult);
        }
    }
}
```

---

### Detailed Error Reporting

```java
package com.example;

import com.github.madhavdhatrak.blaze4j.BlazeValidator;
import com.github.madhavdhatrak.blaze4j.CompiledSchema;
import com.github.madhavdhatrak.blaze4j.SchemaCompiler;
import com.github.madhavdhatrak.blaze4j.ValidationResult;

public class DetailedValidationTest {
    public static void main(String[] args) {
        String enumSchemaJson = """
            {
                "$schema": "https://json-schema.org/draft/2020-12/schema",
                "enum": ["red", "green", "blue"]
            }
            """;

        String invalidEnumInstance = "\"yellow\"";

        SchemaCompiler compiler = new SchemaCompiler();
        try (CompiledSchema compiledSchema = compiler.compile(enumSchemaJson)) {
            BlazeValidator validator = new BlazeValidator();
            ValidationResult result = validator.validateWithDetails(compiledSchema, invalidEnumInstance);

            System.out.println("Is valid: " + result.isValid());

            if (!result.isValid()) {
                System.out.println("Validation errors:");
                result.getErrors().forEach(System.out::println);
            }
        }
    }
}
```

---

### Pre-registered Schema Example

```java
package com.example;

import com.github.madhavdhatrak.blaze4j.BlazeValidator;
import com.github.madhavdhatrak.blaze4j.CompiledSchema;
import com.github.madhavdhatrak.blaze4j.SchemaCompiler;
import com.github.madhavdhatrak.blaze4j.SchemaRegistry;

public class PreRegisterSchemaTest {
    public static void main(String[] args) {
        String integerSchema = """
            {
              "$schema": "https://json-schema.org/draft/2020-12/schema",
              "type": "integer"
            }""";

        // create the registry and register the schema
        SchemaRegistry registry = new SchemaRegistry();
        registry.register("my-integer-schema", integerSchema);

        SchemaCompiler compiler = new SchemaCompiler(registry);

        String mainSchema = """
            {
              "$schema": "https://json-schema.org/draft/2020-12/schema",
              "$ref": "my-integer-schema"
            }""";

        try (CompiledSchema schema = compiler.compile(mainSchema)) {
            BlazeValidator validator = new BlazeValidator();
            boolean validResult = validator.validate(schema, "42");
            System.out.println("validResult: " + validResult);

            boolean invalidResult = validator.validate(schema, "\"not an integer\"");
            System.out.println("invalidResult: " + invalidResult);
        }
    }
}
```

---

## Supported Drafts

| Draft Version | Status      | Specification                                            |
| ------------- | ----------- | -------------------------------------------------------- |
| Draft 2020-12 | ✅ Supported | https://json-schema.org/draft/2020-12/release-notes.html |
| Draft 2019-09 | ✅ Supported | https://json-schema.org/draft/2019-09/release-notes.html |
| Draft 7       | ✅ Supported | https://json-schema.org/draft-07/release-notes.html      |
| Draft 6       | ✅ Supported | https://json-schema.org/draft-06/release-notes.html      |
| Draft 4       | ✅ Supported | https://json-schema.org/draft-04/release-notes.html      |

---

<a name="supported-resolvers"></a>

## Supported `$ref` Resolvers

| Resolver Type | Status      | Description                                |
| ------------- | ----------- | ------------------------------------------ |
| `http`        | ✅ Supported | Resolves external schemas via HTTP(S) URLs |
| `classpath`   | ✅ Supported | Resolves schemas from the Java classpath   |

---

## API Documentation

* [Compiler API](./docs/Compiler.md) — How to compile schemas, manage memory, and use different dialects.
* [Validator API](./docs/Validator.md) — How to validate instances, get boolean or detailed results, and interpret errors.

---

## Best Practices

* Always use try-with-resources for `CompiledSchema` (and `Arena` when used) to guarantee native memory is released.
* Prefer detailed validation (`validateWithDetails`) for debugging and error reporting.
* Always specify a `$schema` or default dialect for maximum compatibility.

---

## Contributing

Feel free to open an issue for bugs or feature requests, or submit a pull request to improve the project. Whether it's fixing a typo, improving the docs, or adding new features—every contribution counts!

For local installation and development, see [CONTRIBUTING.md](./docs/CONTRIBUTING.md). The project uses Git submodules, so when cloning the repository, use:

```bash
git clone --recurse-submodules <repo-url>
```

---

## Credits

* Blaze — Native JSON Schema validator
* JSON Schema Test Suite

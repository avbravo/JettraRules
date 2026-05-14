# JettraRules Guide

JettraRules is a powerful business rule engine for JettraStack that allows you to define complex validations using annotations.

## The @Rules Annotation

The `@Rules` annotation can be applied to fields, methods, or classes to enforce business constraints.

### Parameters

- **field**: Optional field name (used primarily when the annotation is at class level).
- **apply**: The validation operation to perform.
- **than**: The comparison target. It can be another field name or a literal value.
- **message**: Custom error message.

### Supported Operations

| Operation | Description |
|-----------|-------------|
| `equals` | Checks if the value is equal to the target. |
| `notequals` | Checks if the value is NOT equal to the target. |
| `greater` | Checks if the value is strictly greater than the target. |
| `greaterorequals` | Checks if the value is greater than or equal to the target. |
| `less` | Checks if the value is strictly less than the target. |
| `lessorequals` | Checks if the value is less than or equal to the target. |
| `contains` | Checks if the string representation contains the target. |
| `notcontains` | Checks if the string representation does NOT contain the target. |
| `startswith` | Checks if the value starts with the target. |
| `endswith` | Checks if the value ends with the target. |
| `regex` | Validates against a Regular Expression. |

---

## Usage Examples

### Comparison between fields

This is the most common use case, where you want to ensure a field's value depends on another field.

```java
public class FacturaModel {
    private Double saldo;

    @Rules(field="saldo", apply="lessorequals", than="saldo", message="El descuento no puede superar al saldo")
    private Double descuento;
}
```

In this case, `descuento` will be validated against the current value of `saldo`.

### Validation against literal values

```java
public class ProductoModel {
    @Rules(apply="greater", than="0", message="El precio debe ser positivo")
    private Double precio;
}
```

### Regular Expressions

```java
public class UsuarioModel {
    @Rules(apply="regex", than="^[A-Z]{3}-\\d{3}$", message="Formato de código inválido (AAA-999)")
    private String codigoInterno;
}
```

---

## Executing the Rules

To validate an object, use the `JettraRulesEngine`:

```java
FacturaModel factura = new FacturaModel();
factura.setSaldo(100.0);
factura.setDescuento(150.0);

List<RuleResult> results = JettraRulesEngine.validate(factura);

for (RuleResult result : results) {
    if (!result.isValid()) {
        System.out.println("Error en campo " + result.getField() + ": " + result.getMessage());
    }
}
```

## Localización de Mensajes

JettraRules permite el uso de etiquetas de archivos de propiedades (como `messages.properties`) para los mensajes de error.

### Uso en el Modelo

En lugar de escribir un mensaje literal, use la clave que se encuentra en su archivo de propiedades:

```java
public class ProductoModel {
    @Rules(apply="greater", than="0", message="msg.error.precio_positivo")
    private Double precio;
}
```

### Ejecución con Propiedades

Para que el motor resuelva las etiquetas, pase el objeto `Properties` al método `validate`:

```java
// Cargado previamente desde messages.properties
Properties myMessages = ...; 

List<RuleResult> results = JettraRulesEngine.validate(producto, myMessages);
```

Si la clave no se encuentra en el archivo de propiedades, el motor utilizará el valor del atributo `message` como un mensaje literal.

## Integration with JettraWUI

`JettraRules` can be integrated with `CrudView` to provide real-time validation feedback in the UI. When a form is submitted, the engine validates the model and returns notifications to the user if any rule is violated.

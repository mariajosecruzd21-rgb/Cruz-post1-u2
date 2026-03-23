# Creational Patterns - Sistema de Notificaciones

## 📋 Descripción del Proyecto

Este proyecto es una demostración práctica de dos **patrones de diseño creacionales** implementados en Java: **Singleton** y **Factory Method**.

El sistema implementa un gestor centralizado de notificaciones que permite:
- Enviar mensajes a través de múltiples canales (Email, SMS, Push Notifications)
- Registrar y auditar todas las notificaciones en tiempo real
- Agregar nuevos tipos de notificadores sin modificar código existente
- Garantizar una única instancia del logger en toda la aplicación

### Caso de Uso
Un sistema de e-commerce que necesita notificar a sus clientes sobre órdenes de compra a través de diferentes canales de comunicación, manteniendo un registro centralizado de todas las notificaciones enviadas.

---

## 🏗️ Patrones de Diseño Implementados

### 1. **SINGLETON**

#### Problema que resuelve
En aplicaciones complejas, necesitamos garantizar que ciertos recursos (conexiones a BD, loggers, configuraciones globales) existan **en una única instancia** en toda la aplicación. Sin Singleton, corremos el riesgo de:
- Múltiples instancias del logger con registros incompletos
- Inconsistencia en los datos
- Desperdicio de memoria

#### Implementación en el Proyecto
```java
public enum NotificationLogger {
    INSTANCE;
    
    private final List<String> entries = new ArrayList<>();
    
    public void log(String channel, String recipient, String message) {
        // Registra todas las notificaciones
    }
}
```

**Características:**
- **Instancia única garantizada**: Implementado como `enum` (enfoque más seguro)
- **Thread-safe por defecto**: La JVM garantiza que los enums son singleton seguros
- **Sin reflexión**: No hay riesgo de crear múltiples instancias mediante reflection
- **Acceso global**: `NotificationLogger.INSTANCE` desde cualquier punto del código

**Beneficio en el proyecto:** 
Todos los notificadores (EmailNotifier, SmsNotifier, PushNotifier) usan la misma instancia del logger para registrar, asegurando un historial completo y consistente de todas las notificaciones.

---

### 2. **FACTORY METHOD**

#### Problema que resuelve
Cuando el código cliente necesita crear objetos de clases desconocidas en tiempo de compilación, Factory Method nos ayuda a:
- Desacoplar la creación de objetos de su uso
- Facilitar la adición de nuevos tipos sin modificar código existente (OCP)
- Centralizar la lógica de instantiación
- Permitir registro dinámico de nuevos tipos en tiempo de ejecución

#### Implementación en el Proyecto
```java
public class NotifierFactory {
    private static final Map<String, Supplier<Notifier>> REGISTRY = 
        new HashMap<>();
    
    static {
        REGISTRY.put("email", EmailNotifier::new);
        REGISTRY.put("sms", SmsNotifier::new);
        REGISTRY.put("push", PushNotifier::new);
    }
    
    public static Notifier create(String type) {
        Supplier<Notifier> factory = REGISTRY.get(type.toLowerCase());
        if (factory == null) {
            throw new IllegalArgumentException(...);
        }
        return factory.get();
    }
    
    public static void register(String type, Supplier<Notifier> factory) {
        REGISTRY.put(type.toLowerCase(), factory);
    }
}
```

**Características:**
- **Registro dinámico**: Nuevos tipos se pueden agregar en tiempo de ejecución
- **Extensibilidad**: Sin modificar la factory, se pueden agregar nuevos notificadores
- **Type-safe**: Usa `Supplier<Notifier>` para mantener tipado
- **Case-insensitive**: Los tipos se normalizan a minúsculas

**Beneficio en el proyecto:**
El cliente no necesita conocer las clases concretas (EmailNotifier, SmsNotifier, etc.). Solo llama a:
```java
Notifier email = NotifierFactory.create("email");
```

Incluso se pueden registrar nuevos tipos en tiempo de ejecución:
```java
NotifierFactory.register("slack", () -> new Notifier() { ... });
```

---

## 📁 Estructura del Proyecto

```
creational-patterns/
├── pom.xml                                 # Configuración Maven
├── README.md                               # Este archivo
└── src/main/java/com/patrones/u2/
    ├── Main.java                           # Clase principal con demo
    ├── Notifier.java                       # Interfaz (Product)
    ├── NotificationLogger.java             # Singleton (enum)
    ├── NotifierFactory.java                # Factory Method
    ├── EmailNotifier.java                  # Implementación concreta
    ├── SmsNotifier.java                    # Implementación concreta
    └── PushNotifier.java                   # Implementación concreta
```

---

## 🚀 Instrucciones de Ejecución

### Requisitos Previos
- **Java 25+** (configurado en el `pom.xml`)
- **Maven 3.6+**

### Pasos para Ejecutar

#### Opción 1: Usando Maven desde la terminal

1. **Navegar al directorio del proyecto:**
   ```bash
   cd /Users/mariajosecruzduarte/IdeaProjects/creational-patterns
   ```

2. **Compilar el proyecto:**
   ```bash
   mvn clean compile
   ```

3. **Ejecutar la clase principal:**
   ```bash
   mvn exec:java -Dexec.mainClass="com.patrones.u2.Main"
   ```

#### Opción 2: Usando IntelliJ IDEA

1. Abre el proyecto en IntelliJ IDEA
2. Navega hasta: `src/main/java/com/patrones/u2/Main.java`
3. Haz clic derecho en la clase → **Run 'Main.main()'**
4. O presiona: `Ctrl+Shift+R` (macOS: `^Shift+R`)

#### Opción 3: Compilar y ejecutar manualmente

1. **Compilar:**
   ```bash
   javac -d target/classes src/main/java/com/patrones/u2/*.java
   ```

2. **Ejecutar:**
   ```bash
   java -cp target/classes com.patrones.u2.Main
   ```

---

## 📊 Salida Esperada

Al ejecutar la aplicación, verás:

```
=== Demo: Singleton + Factory Method ===

Misma instancia: true

[2026-03-23 14:30:45] [EMAIL] -> cliente@mail.com: Su pedido #1001 fue confirmado
[2026-03-23 14:30:45] [SMS] -> +57-300-0000001: Pedido #1001 confirmado
[2026-03-23 14:30:45] [PUSH] -> device-token-abc123: Nuevo pedido listo para enviar
[2026-03-23 14:30:45] [SLACK] -> #pedidos: Pedido #1001 procesado

=== Historial de Notificaciones ===
[2026-03-23 14:30:45] [EMAIL] -> cliente@mail.com: Su pedido #1001 fue confirmado
[2026-03-23 14:30:45] [SMS] -> +57-300-0000001: Pedido #1001 confirmado
[2026-03-23 14:30:45] [PUSH] -> device-token-abc123: Nuevo pedido listo para enviar
[2026-03-23 14:30:45] [SLACK] -> #pedidos: Pedido #1001 procesado
Total: 4 notificaciones
```

---

## 🔑 Conceptos Clave Demorados

| Concepto | Descripción |
|----------|------------|
| **Singleton** | Una única instancia global y controlada |
| **Factory Method** | Creación de objetos sin especificar clases concretas |
| **OCP (Open/Closed Principle)** | Abierto a extensión, cerrado a modificación |
| **Dependency Inversion** | Depender de abstracciones, no de implementaciones |
| **Registry Pattern** | Registro dinámico de tipos en tiempo de ejecución |

---

## 💡 Ventajas de Esta Implementación

✅ **Desacoplamiento**: El código cliente no conoce implementaciones concretas  
✅ **Extensibilidad**: Agregar nuevos notificadores es trivial  
✅ **Centralized Logging**: Un único punto de auditoría  
✅ **Thread-safe**: El Singleton enum es inherentemente seguro para hilos  
✅ **Flexible**: Registro dinámico de nuevos tipos sin recompilación  
✅ **Mantenible**: Código limpio y fácil de entender  

---

## 📚 Referencias

- [Singleton Pattern](https://refactoring.guru/design-patterns/singleton)
- [Factory Method Pattern](https://refactoring.guru/design-patterns/factory-method)
- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)
- [Design Patterns in Java](https://www.baeldung.com/design-patterns-in-java)

---

## 👤 Autor

Proyecto desarrollado como parte del curso de Patrones de Diseño Creacionales.

---

**Última actualización:** 23 de marzo de 2026


# 🧠 Event-Driven Architecture con RabbitMQ y Outbox

Este documento resume el flujo completo de eventos implementado en **Agenda Manager**, combinando todo el trabajo realizado durante los últimos días. Se trata de una arquitectura robusta basada en **Transactional Outbox + RabbitMQ + MediatR**, diseñada para tolerancia a fallos, trazabilidad y escalabilidad real.

---

## 📦 Flujo General del Sistema

1. **Dominio lanza eventos (`IDomainEvent`)**
   - Se agregan a los agregados raíz durante la lógica de dominio.

2. **Interceptor `PersistDomainEventsToOutbox`**
   - Extrae los eventos del `DbContext` y los guarda como `OutboxMessage` (serializados con Newtonsoft).

3. **HostedService `OutboxMessageProcessorHostedService`**
   - Lee periódicamente los eventos `Pending` en Outbox.
   - Publica los eventos al `Exchange` de RabbitMQ mediante `IRabbitMqClient`.
   - Marca cada mensaje como `Processed` o `Failed`.

4. **RabbitMQ Exchange y Queue**
   - El mensaje se enruta usando el `routingKey` (nombre del evento) hacia la cola `agenda.event.queue`.

5. **HostedService `RabbitMqConsumerHostedService`**
   - Escucha la cola y recibe mensajes publicados.
   - Llama a `IntegrationEventDispatcher` para procesarlos.

6. **`IntegrationEventDispatcher`**
   - Obtiene el tipo de evento desde `routingKey`.
   - Deserializa el payload usando Newtonsoft.
   - Invoca el handler correcto a través de `IMediator.Publish`.

7. **Handlers de aplicación (`INotificationHandler<T>`)**
   - Ejecutan la lógica correspondiente al evento.

---

## 🏗 Diagrama de flujo

```
DomainEvent  →  OutboxMessage
                ↓
     PersistDomainEventsToOutbox (Interceptor)
                ↓
     OutboxMessageProcessorHostedService
                ↓
         RabbitMQ Exchange
                ↓
           RabbitMQ Queue
                ↓
     RabbitMqConsumerHostedService
                ↓
     IntegrationEventDispatcher
                ↓
     INotificationHandler<T>
```

---

## 🛠 Componentes Técnicos Importantes

- `OutboxMessages` tiene estados (`Pending`, `Processed`, `Failed`) → trazabilidad
- `UserTokenCreatedDomainEvent` y similares son eventos ligeros (solo contienen `Id` y datos mínimos)
- `XxxxId` son ValueObjects con `internal` constructor + `[JsonConstructor]` para evitar problemas de deserialización

Ejemplo:

```csharp
internal record UserTokenCreatedDomainEvent(UserTokenId UserTokenId) : IDomainEvent;
```

---

## 📁 Configuración básica

### `appsettings.json`

```json
"RabbitMq": {
  "Host": "localhost",
  "Port": 5672,
  "User": "guest",
  "Password": "guest",
  "Exchange": "agenda.exchange",
  "QueueName": "agenda.event.queue"
}
```

---

## 📈 Trazabilidad y auditoría

- RabbitMQ **no guarda histórico de eventos ya procesados**
- El sistema usa `OutboxMessages` como fuente de auditoría
- Esto permite ver qué evento falló, cuál está pendiente o procesado

---

## ✅ Ventajas del nuevo enfoque

- Alta tolerancia a fallos
- Flujo desacoplado y asincrónico
- Reintentos posibles
- Trazabilidad centralizada
- Facilidad para escalar y distribuir responsabilidades

---

## 💡 Lecciones aprendidas

- El patrón Outbox resuelve elegantemente la asincronía
- Desacoplar los handlers del flujo directo mejora el mantenimiento
- Rabbit es un pipe, no un histórico: Outbox cumple ese rol

---

## 📍 Mejoras futuras (to-do)

- Retries con delay
- DLQ (Dead Letter Queue) para fallos críticos
- Testear más a fondo flujos edge-case
- Dividir colas por tipo de evento
- Monitorización automática del estado de las colas

---

## 🏁 Resultado Final

Sistema robusto, resiliente y documentado.
Diseñado para aguantar escenarios reales y crecer sin comprometer la arquitectura limpia.

Sí, el sistema ha pasado de MVP optimista a infraestructura profesional. 🎯

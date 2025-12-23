# Лабораторная работа №3

**Тема:** Использование принципов проектирования на уровне методов и классов  
**Цель работы:** Получить опыт проектирования и реализации модулей с использованием принципов KISS, YAGNI, DRY, SOLID и др.

## Диаграмма контейнеров

![container_diagram_.png](container_diagram_.png)

## Диаграмма компонентов

![user_service_component_diagram.png](user_service_component_diagram.png)

## Диаграмма последовательностей

Use-case регистрации - в два этапа. Сначала запрашивается код, потом вводится.
![user_registration_sequence.png](user_registration_sequence.png)

## Модель БД

![database_model.png](database_model.png)

## Применение основных принципов разработки

### KISS (Keep It Simple, Stupid)

**Принцип простоты** — код должен быть максимально простым и понятным, без излишней сложности.

```csharp
// PhoneVerificationService.cs
public class PhoneVerificationService
{
    private readonly Random _random = new Random();

    public string GenerateRandomCode()
    {
        return _random.Next(100000, 999999).ToString();
    }


    public bool IsCodeValid(PhoneVerification verification)
    {
        return !verification.IsUsed
               && verification.ExpiresAt > DateTime.UtcNow
               && verification.AttemptsCount < 3;
    }
}
```

**Применение KISS:** Методы выполняют одну простую задачу без излишних абстракций. `GenerateRandomCode()` генерирует 6-значное число без сложных алгоритмов. `IsCodeValid()` проверяет три простых условия в читаемом виде — код не использован, не истек и не превышен лимит попыток.

## Дополнительные принципы разработки

<По каждому принципу разработчик из раздела повышенной сложности обосновать отказ или применение>

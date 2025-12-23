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

### YAGNI (You Aren't Gonna Need It)

**Принцип "Вам это не понадобится"** — не реализовывать функциональность, которая не нужна прямо сейчас.

```csharp
// AuthService.cs
public class AuthService
{
    private readonly IUserRepository _userRepository;
    private readonly IPasswordHasher _passwordHasher;

    public async Task<User> RegisterAsync(RegisterRequest request)
    {
        // Проверяем только то, что нужно сейчас - существует ли пользователь
        var existingUser = await _userRepository.GetByPhoneAsync(request.Phone);
        if (existingUser != null)
            throw new UserAlreadyExistsException();

        // Создаем пользователя с минимальным набором полей
        var user = new User
        {
            Phone = request.Phone,
            PasswordHash = _passwordHasher.HashPassword(request.Password),
            FirstName = request.FirstName,
            LastName = request.LastName,
            IsVerified = false
        };

        return await _userRepository.CreateAsync(user);
    }
}
```

**Применение YAGNI:**
- **Не добавлена** сложная валидация email (его еще нет в системе)
- **Не реализован** механизм множественных попыток регистрации с разными номерами
- **Не создана** система профилей с аватарами, биографией и другими полями "на будущее"
- **Не добавлен** функционал смены номера телефона до верификации

Реализовано только то, что необходимо для базовой регистрации: проверка уникальности телефона, хеширование пароля и сохранение минимального набора данных.

### DRY (Don't Repeat Yourself)

**Принцип "Не повторяйся"** — избегать дублирования кода, выносить общую логику в переиспользуемые компоненты.

```csharp
// BaseRepository.cs - базовый репозиторий для избежания дублирования
public abstract class BaseRepository<T> where T : class
{
    protected readonly DbContext _context;

    protected BaseRepository(DbContext context)
    {
        _context = context;
    }

    // Общий метод создания - используется всеми репозиториями
    public async Task<T> CreateAsync(T entity)
    {
        await _context.Set<T>().AddAsync(entity);
        await _context.SaveChangesAsync();
        return entity;
    }

    // Общий метод получения по ID
    public async Task<T?> GetByIdAsync(Guid id)
    {
        return await _context.Set<T>().FindAsync(id);
    }
}

// UserRepository.cs - наследует общую логику
public class UserRepository : BaseRepository<User>, IUserRepository
{
    public UserRepository(DbContext context) : base(context) { }

    // Только специфичная логика для User
    public async Task<User?> GetByPhoneAsync(string phone)
    {
        return await _context.Set<User>()
            .FirstOrDefaultAsync(u => u.Phone == phone);
    }
}

// RefreshTokenRepository.cs - также использует базовую логику
public class RefreshTokenRepository : BaseRepository<RefreshToken>, IRefreshTokenRepository
{
    public RefreshTokenRepository(DbContext context) : base(context) { }

    // Только специфичная логика для RefreshToken
    public async Task<RefreshToken?> GetValidTokenAsync(string tokenHash)
    {
        return await _context.Set<RefreshToken>()
            .FirstOrDefaultAsync(t => t.TokenHash == tokenHash
                                   && t.ExpiresAt > DateTime.UtcNow
                                   && t.RevokedAt == null);
    }
}
```

**Применение DRY:**
- Общие методы `CreateAsync()` и `GetByIdAsync()` вынесены в базовый класс `BaseRepository<T>`
- Избегаем дублирования кода работы с `DbContext` в каждом репозитории
- Специфичная логика (поиск по телефону, поиск валидного токена) реализована только там, где нужна
- При необходимости изменить логику создания или получения сущности — правим в одном месте



## Дополнительные принципы разработки

<По каждому принципу разработчик из раздела повышенной сложности обосновать отказ или применение>

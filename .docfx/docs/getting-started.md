# Getting Started

### Instalation

Install MiniMapr via NuGet:

```bash
dotnet add package MiniMapr
```

### Defining Source and Target Classes
Let's assume you have a domain model and a DTO:

```csharp
public class User
{
    public string Name { get; set; }
    public string Email { get; set; }
    public string Password { get; set; }
    public Address Address { get; set; }
}

public class UserDto
{
    public string Name { get; set; }
    public string EmailAddress { get; set; }
    public string Password { get; set; } = "";
    public AddressDto Address { get; set; }
}

public class Address
{
    public string Street { get; set; }
}

public class AddressDto
{
    public string Street { get; set; }
}
```

### Creating Mapping Configuration Classes
To define how mapping should occur, implement IMapperConfig:

Example:
```csharp
using MiniMapr;

public class UserMappingConfig : IMapperConfig 
{ 
    public void Configure(MapperService mapper) 
    { 
        mapper.Add<User, UserDto>(config => 
        { 
            config.IgnoredProperties.Add("Password"); 
            config.CustomMappings["Email"] = "EmailAddress"; 
            config.AddCustomTransformation<Address, AddressDto>("Address", address =>
                mapper.Map<Address, AddressDto>(address));
        });
    }
}

public class AddressMappingConfig : IMapperConfig 
{ 
    public void Configure(MapperService mapper) 
    { 
        mapper.Add<Address, AddressDto>(); 
    } 
}
```
### Registering MiniMapr
To enable MiniMapr in your application, register your mapping configurations:

```csharp
var services = new ServiceCollection();

services.AddMapper(mapper => 
{ 
    mapper.Add<UserMappingConfig>()
        .Add<AddressMappingConfig>();
});

var serviceProvider = services.BuildServiceProvider();
var mapperService = serviceProvider.GetRequiredService<IMapper>();
```

### Performing a Mapping
Once registered, you can use the mapper as follows:

```csharp
var user = new User
{
    Name = "Alice",
    Email = "alice@email.com",
    Password = "secret123",
    Address = new Address { Street = "123 Maple St" }
};

var userDto = mapperService.Map<User, UserDto>(user);

Console.WriteLine(userDto.Name); // Alice
Console.WriteLine(userDto.EmailAddress); // alice@email.com
Console.WriteLine(userDto.Password); // (empty)
Console.WriteLine(userDto.Address.Street); // 123 Maple St
```

Now you're ready to build fast, customizable object mappings with MiniMapr!
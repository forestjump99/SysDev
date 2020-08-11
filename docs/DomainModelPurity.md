# Domain model purity

"Domain model purity" är när domänmodellerna inte anropar några "out-of-process dependencies", dvs databaser, filsystem, etc. Domänklasser ska bara använda sig av primitiva typer och andra domänklasser.

## Exempel - Ändra användarens e-postadress och verifiera att den inte redan existerar.
Kontrollen av att e-postadressen är unik (i en databas) sker i UserController. Detta gör att vi att Domänmodellen uppfyller "Domain model purity", men ej "Domain model completeness"

```csharp
public class User : Entity
{
    public Company Company { get; private set; }
    public string Email { get; private set; }

    public Result ChangeEmail(string newEmail)
    {
        if (Company.IsEmailCorporate(newEmail) == false)
            return Result.Failure("Incorrect email domain");

        Email = newEmail;

        return Result.Success();
    }
}

public class Company : Entity
{
    public string DomainName { get; }

    public bool IsEmailCorporate(string email)
    {
        string emailDomain = email.Split('@')[1];
        return emailDomain == DomainName;
    }
}
```
Och det här är Controllern som orkestrerar anropen:

```csharp
// UserController
public string ChangeEmail(int userId, string newEmail)
{
    /* The new validation */
    User existingUser = _userRepository.GetByEmail(newEmail);
    if (existingUser != null && existingUser.Id != userId)
        return "Email is already taken";

    User user = _userRepository.GetById(userId);

    Result result = user.ChangeEmail(newEmail);
    if (result.IsFailure)
        return result.Error;

    _userRepository.Save(user);

    return "OK";
}
```
